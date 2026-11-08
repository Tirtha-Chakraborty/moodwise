# V1 TDD

**Status:** Draft in progress (2026-11-04). The TDD is organised by topic, named as in [tdd_outline.md](tdd_outline.md), which tracks progress. Written so far: the overview (awaiting review), tech stack, request flow (awaiting review), architecture and components, the taxonomy and song links in the data model, hosting in config and deployment, and the frontend and design pass decisions. Sections and parts marked TBD are not written yet. Each topic holds its design and its decisions, and each decision gives the decision itself, the rejected alternative, and what breaks if it's wrong. Every decision also has a row in [decision_logs.md](../decision_logs.md). First draft in Week 1, finalized in Week 2.

## Overview, goals and non-goals

Moodwise V1 takes a phrase, classifies its mood with an LLM, and returns songs and quotes from a hand-curated dataset. The problem, audience, scope and success criteria are in the [PRD](prd.md); this document says how V1 is built.

**Constraints**

- Go
- Fixed mood buckets, classified by an LLM API
- Hand-curated flat JSON dataset
- Server-rendered frontend and backend in the same codebase
- No embeddings and no database

**Non-goals:** books and pictures, embeddings, external data sources, caching, a database, accounts and personalization, and full rate limiting (V4). See the PRD scope.

## Tech stack

| Area | Choice | Notes |
| --- | --- | --- |
| Language | Go 1.27, set in `go.mod` | Latest stable at the time of writing (1.27.3, with 1.26 as the previous release). Move to a newer stable release between weeks, not mid-week. |
| HTTP server and router | Standard library `net/http` with `ServeMux` | Method and path patterns such as `POST /api/match` are built in. Three routes do not need a router library. |
| Pages | `html/template`, server-rendered; templates and static assets embedded with `go:embed` | Escapes output by default. One deployable. See "Frontend and design system". |
| Interactivity | htmx, vendored at a pinned version in `web/static/`, plus small vanilla JS | Loading state and result swaps; in V3, streamed results over Server-Sent Events. Never loaded from a CDN. Not a Go module dependency. |
| Styling | Design tokens as CSS variables in one file; the CSS approach is chosen before the first page is styled | See "Frontend and design system". |
| API | A JSON API next to the pages (`POST /api/match`) | Keeps the backend reusable by another frontend later. Route details are in "Routes and API contract". |
| Logging | `log/slog`, structured | Fields are decided in "Observability". |
| Configuration | Environment variables, read and validated once at startup | A missing or invalid value stops the app from booting. |
| Taxonomy and dataset | JSON files, validated by the loader at startup | The taxonomy is embedded in the binary; the dataset's location is decided in "Data model". |
| LLM | Gemini Flash-Lite, OpenAI as fallback | See "Decision: LLM provider and model" below. |
| Tests | Standard library `testing`, table-driven, `net/http/httptest`, `-race` from Week 5 | No assertion library. |
| Static checks | `go vet` and a `gofmt` check, both in CI | No linter suite in V1. |
| Build and tasks | A Makefile wrapping the Go commands; CI runs the same targets | Developers need `make` installed locally. |
| Container | Multi-stage Dockerfile, non-root user | Built in Week 6. |
| Hosting | Not chosen yet | Deferred to Week 6 (Q15). See "Config and deployment". |
| Go module dependencies | The LLM provider SDK only, if one is used | V1 only. Anything else needs a reason written down first. Later versions add libraries with a decision row at the start of each (see "Decision: architecture shape and long-term stack" below). |

### Decision: architecture shape and long-term stack

- **Decision:** one Go module, one repo and one deployable binary: a modular monolith. Later categories and sources are packages behind interfaces, not separate services (`Classifier` in V1, `Matcher` and `Embedder` in V2, `Source` in V3; the layout is in [repo_structure.md](../repo_structure.md)). Go stays for V1 to V4. Python is revisited only if the project needs local models or a Python-only library, and then as one small service behind an interface. The V2 to V4 libraries are not chosen now; each is chosen at the start of its version. The likely picks are forecasts, not decisions: V2 precomputed embeddings with in-memory cosine similarity and an in-process TTL cache; V3 Postgres with `pgx`, `sqlc`, `goose` and `pgvector`, `errgroup`, a hand-written retry and circuit breaker, and Prometheus metrics; V4 cookie sessions stored in Postgres, `x/time/rate`, and `chi` if per-route middleware gets messy.
- **Rejected alternative:** microservices; Python for the main service; a separate repo for the frontend; choosing the V3 and V4 libraries now.
- **What breaks if wrong:** a monolith whose packages import each other freely becomes tangled, so the dependency rules in the repo structure plan matter. If V2 needs a Python-only library, the cost is a small service behind the `Matcher` or `Embedder` interface, which is why those interfaces exist. The forecast picks may turn out wrong, which is why they are not binding.

### Decision: standard library first

- **Decision:** use the Go standard library for the router (`net/http` `ServeMux`), tests (`testing` and `httptest`), templates and logging, and keep the third-party dependency list as short as possible. Static checks in CI are `go vet` and a `gofmt` check. Use the latest stable Go release. These rules are V1-scoped and cover Go module dependencies: later versions add libraries when they need them (see the architecture decision above).
- **Rejected alternative:** a router or web framework (chi, gin, echo); an assertion library such as testify; a lint suite such as golangci-lint in V1; staying on an older Go release.
- **What breaks if wrong:** if the app grows route groups or per-route middleware chains, plain `ServeMux` means writing a small helper, or adopting chi, which is compatible with `http.Handler` and cheap to add later. A smaller toolbox means fewer lint rules catching style or bug classes early, so code review stays the safety net in V1. A newest-release Go can lag in tool and image support, so the toolchain version is pinned in `go.mod` and the container build.

### Decision: LLM provider and model

- **Decision:** the classifier starts on Google's Gemini API with a Flash-Lite model, with OpenAI as the planned fallback. The exact model ID is a Week 3 config value, chosen from Google's current docs and confirmed by the eval. The fallback OpenAI model is chosen at pivot time by running the eval, not named now. Cost per request is a fraction of a cent for any of the candidates (about 500 input and 15 output tokens), so cost does not decide between them. Moving up within a provider (Flash-Lite to Flash) is the first escalation if the eval misses its targets; switching provider is the second.

  The design keeps a provider pivot cheap:
  - Everything provider-specific (SDK, request shape, structured-output setup, auth, error mapping) sits in one package behind the classifier boundary. Nothing else imports a provider SDK.
  - The prompt text, the taxonomy and the enum validation are provider-neutral and live outside that package.
  - Provider and model name come from config, so a model swap within a provider needs no code change.
  - Only one provider feature is used: a JSON-schema-constrained single-value output. No provider-specific tools, caching or thinking features.
  - Provider-specific failures (safety blocks, refusals, truncated output, timeouts) are mapped inside the package to the same small set of classifier errors, so the handler's behavior does not depend on the provider.
  - Thinking or reasoning is kept minimal, since extra thinking tokens add cost and latency on a task with a one-word answer.
  - The eval runs against whichever provider is configured, so a pivot is checked by the same 90% and 100% targets.
- **Rejected alternative:** Anthropic Haiku 5.5 as the starting model; OpenAI first; a fine-tuned open model such as `j-hartmann/emotion-english-distilroberta-base` (7 fixed labels that do not match the taxonomy, no `unclear` class, and it needs a Python runtime beside the Go service); a larger model from the start (no cost reason, and slower).
- **What breaks if wrong:** if Flash-Lite misses the eval, the response is a model upgrade, then a provider pivot, estimated at 3-5 hours for a new adapter, prompt re-tuning and an eval run (an estimate, not measured). Provider safety filters may block ordinary grief or heartbreak phrases, which would show up as wrongly empty results unless they are mapped to the `unclear` message. Pricing and model names change often, and the figures above came from third-party aggregators, so the real ID and price are checked against the provider's own pages before the Week 3 build. If SDK calls leak outside the one package, a pivot grows from hours into a refactor.

## Architecture and components

V1 is one Go service. It renders pages and answers a JSON API from the same code, and it calls one external service, the LLM provider. Diagrams are in [design/architecture.md](design/architecture.md).

### Components

| Component | Job | Must not know about |
| --- | --- | --- |
| Server (handlers) | Reads the request, validates the input, calls the `Matcher`, turns its outcome into a view model, renders it as a page or a JSON response, and serves `/healthz` | How matching works, the LLM, the dataset |
| Middleware | Wraps every request around the handler for cross-cutting concerns: request ID, access logging (never the phrase), panic recovery, security headers and timeouts. The spend guard is not middleware: it runs after input validation, just before the paid LLM call (see "Request flow") | Matching, the dataset |
| Templates and web assets | Turn a view model into a page or an htmx fragment; embedded in the binary and parsed once at startup | Domain types, how the outcome was produced |
| Config | The values that differ per environment (the API key, provider and model, port, LLM timeout, input length cap, spend limit, dataset location), read and validated once at startup and passed in. The final list is in "Config and deployment" | The components that consume the values |
| `Matcher` (interface) | Phrase in, outcome out. The outcome is a result (detected mood plus songs and quotes), the `unclear` outcome, or a typed failure | HTTP, templates |
| Bucket matcher (the V1 `Matcher`) | Asks the `Classifier` for a mood, then asks the catalog for items in that mood | HTTP, which provider |
| `Classifier` (interface) | Phrase in, a mood key or `unclear` out, or a typed failure | Which provider, HTTP |
| LLM classifier (the V1 `Classifier`) | Builds the prompt from the taxonomy, calls the provider adapter, validates the answer against the taxonomy keys | SDK types |
| Provider adapter (Gemini) | The only code that imports an LLM SDK: request shape, auth, mapping provider errors to the classifier's typed failures | Prompt text, the taxonomy |
| Taxonomy (embedded file, loaded once) | The ten moods with key, label and definition, loaded and validated once at startup into an immutable value that the classifier, the loader and the templates receive | HTTP, the LLM |
| `Catalog` (interface) | Holds the items indexed by mood and offers retrieval by mood key: a random sample of songs and quotes, using an injected random source. The interface is defined where the matcher uses it | Classification, HTTP |
| Startup loader | Reads and validates the taxonomy and dataset and builds the catalog; any error stops boot | HTTP |
| Composition root (`cmd/moodwise`) | Reads config, creates the concrete components, wires them together, runs the server and shuts it down | Nothing is hidden from it, and nothing else does this job |

The exact fields of the outcome and the typed failures are decided in "Request flow", "Failure modes" and "Classifier design".

### Dependency direction

- The server depends on the `Matcher` interface and the domain types, nothing else.
- Implementations depend on the domain types, and the domain types import nothing.
- Only the provider adapter imports an LLM SDK.
- Only the composition root knows which concrete type sits behind an interface.

The package layout that follows from this is in [repo_structure.md](../repo_structure.md).

### Seams for later versions

| Version | What changes | Where it plugs in |
| --- | --- | --- |
| V2 | Embedding matching replaces mood buckets | A second `Matcher` implementation. The server is untouched. `Classifier` may shrink to supplying the displayed mood label, or be retired |
| V2 | Caching, live APIs | A cache that wraps a `Matcher` or a source; a `Source` interface for live APIs, with the catalog becoming a static source |
| V3 | Parallel sources, retries, circuit breaker, a database, metrics | Wrappers that implement the same interface as what they wrap; a store behind its own interface; a streaming response added to the server |
| V4 | Auth, rate limiting, personalization | Middleware in the server; personalization as a new component beside the matcher |

### Decision: the handler depends on one `Matcher`

- **Decision:** the server calls a single `Matcher` interface: phrase in, outcome out. V1's implementation asks the `Classifier` for a mood and then looks up that mood in the catalog. The server never calls the classifier or the catalog itself, and it never learns that a mood sits in the middle.
- **Rejected alternative:** the handler orchestrates the steps itself: call the `Classifier`, then the `Catalog`, then render.
- **What breaks if wrong:** "classify, then look up the bucket" is a V1 detail. V2 matches by embeddings, with no mood in the middle, so a handler that orchestrates those steps has to be rewritten in V2, together with its tests. The cost of the chosen design is one extra interface in V1, and the risk that `Matcher` grows into a catch-all, so its outcome type stays small and carries no provider detail.

### Decision: `Classifier` is an interface (Q2)

- **Decision:** `Classifier` is an interface with two implementations from the start: the LLM classifier and a fake used in tests. Everything provider-specific sits behind it, in the provider adapter. The classifier has two test seams on purpose: a fake `Classifier` for matcher, server and in-process end-to-end tests, and a fake provider client (or a fake LLM server) for tests of prompt building, parsing and enum validation. "Classifier design" covers the second seam.
- **Rejected alternative:** a concrete classifier type that the matcher (or the handler) uses directly, with the SDK call inside it.
- **What breaks if wrong:** without the interface, every matcher test needs the real endpoint, so it costs money, runs slowly and varies between runs, and it cannot make the classifier return `unclear`, time out or return an off-list answer on demand. A provider switch then edits the matcher and every place SDK types leak, instead of one adapter. If the handler called the SDK directly, as Q2 was first phrased, it would also build prompts, parse provider responses and map provider errors, so HTTP and LLM logic would be tangled and a handler test would need the LLM. The interface costs one indirection, and a fake that behaves unlike the real classifier can hide bugs, which is why the eval runs against the real classifier and the end-to-end test uses a fake LLM server.

### Decision: the taxonomy is loaded once and passed in

- **Decision:** the taxonomy file is read and validated once at startup into an immutable value. The classifier (for the prompt and the answer check), the dataset loader (for tag validation) and the templates (for labels) receive that value. None of them reads the file itself.
- **Rejected alternative:** each consumer reading the file itself.
- **What breaks if wrong:** separate reads can disagree. Validation ends up repeated, or skipped in one place. A bad file fails late and unevenly instead of once at boot. Tests have to write files instead of passing in a small taxonomy.

### Decision: `Catalog` is an interface

- **Decision:** `Catalog` holds the items indexed by mood and offers retrieval by mood key. The matcher depends on the interface, which is defined in the matcher's package, where it is used. The mapping is the implementation's private detail.
- **Rejected alternative:** the matcher using a concrete catalog type, or reading the dataset itself.
- **What breaks if wrong:** matcher tests need a loaded dataset and cannot easily force a thin or empty bucket. In V2 the catalog becomes a static source behind a `Source` interface, and a concrete dependency would make that move touch the matcher. The cost is a second interface with one real implementation in V1, which the test fake and the V2 move justify.

### Decision: templates get view models and are parsed once

- **Decision:** the server turns the matcher's outcome into a view model, a small struct holding exactly what the page shows (mood label, song cards, quote cards, messages). Templates contain display logic only and never see domain types. All templates are parsed once at startup. The same view model can feed a full page or an htmx fragment.
- **Rejected alternative:** passing domain types straight to templates; parsing templates on each request.
- **What breaks if wrong:** templates coupled to domain types need editing whenever the domain changes, and they invite decisions into the template, which weakens "handlers render, they do not decide". Parsing on each request is slower, and a syntax error shows up as a 500 on the first request that uses the template instead of stopping boot. The cost is a mapping step from outcome to view model, which needs its own tests.

### Decision: cross-cutting concerns live in middleware

- **Decision:** request ID, access logging, panic recovery, security headers and timeouts are middleware wrapped around the handlers, not code inside them. The order is request ID, then logging, then panic recovery, then the handler. The spend guard is not in this chain: it runs after input validation, directly before the paid LLM call, so a request rejected as bad input never counts against the budget (see "Request flow"; the mechanism and exact placement are settled in "Spend guard"). Access logs record method, path, status and duration, never the phrase.
- **Rejected alternative:** each handler doing its own logging, recovery and guarding.
- **What breaks if wrong:** every new route has to remember every concern, and the route that forgets goes unlogged, unguarded, or crashes the server on a panic. A wrong order fails quietly: a logger outside the request ID middleware loses the ID. A spend guard placed after the LLM call guards nothing, and one placed before validation spends budget on requests that cost nothing, which is why it is not middleware. The cost is that behaviour is spread across wrappers, so the order is written down here.

### Decision: config is read once and passed in, and one composition root wires everything

- **Decision:** configuration is read from environment variables and validated once at startup into a value. No other code reads the environment. The composition root (`cmd/moodwise`) creates every concrete component, passes each the config values it needs, and wires them together. It is the only place that knows which implementation sits behind each interface.
- **Rejected alternative:** components reading environment variables where they need them; package-level singletons, or components that build their own dependencies.
- **What breaks if wrong:** hidden environment reads make behaviour depend on things a test did not set up, and a missing value fails at first use instead of at boot. Components that build their own dependencies cannot be given fakes, which undoes the `Classifier` and `Catalog` decisions. The cost is that the composition root grows with the app, though it is also the one place where all the wiring is visible.

## Request flow

One visitor request, from the browser to the response. The page path and the JSON path (`POST /api/match`) share steps 1 to 6 and differ only in how step 7 renders. The route shapes are in "Routes and API contract".

### Steps

| # | Step | Where | Can fail with |
| --- | --- | --- | --- |
| 1 | The browser posts the phrase | Browser to server | Network error (nothing reaches us) |
| 2 | Request ID, access log, panic recovery, security headers | Middleware | A panic, which becomes a generic error page |
| 3 | Validate the input: trim, reject empty, over-length or invalid text | Handler | Bad input, answered before anything costs money |
| 4 | The handler derives a deadline from the request context and calls the `Matcher` | Handler to `Matcher` | None |
| 5 | The spend guard checks the budget, then the `Classifier` calls the LLM and checks the answer against the taxonomy keys | Guard, classifier, provider adapter | Budget used up, timeout, provider error or safety block, off-list answer |
| 6 | For a mood, the `Catalog` returns a random sample: up to 3 songs and up to 2 quotes | `Matcher` to `Catalog` | None; a thin or empty bucket is a normal result |
| 7 | The outcome becomes a view model (page) or a JSON body, and is rendered | Handler, templates | A template error, caught at startup because templates are parsed once |

The outcome of step 5 and 6 is one of three things:
- **A result:** the detected mood and its items, possibly fewer than 3 and 2, or none in one section.
- **`unclear`:** the model judged that the phrase has no mood. This is an expected outcome, shown as the friendly "try describing how you feel" message.
- **A typed failure:** budget used up, timeout, provider error, or invalid output. The handler maps each kind to a status code and a visitor message without looking at provider detail. The exact list is in "Failure modes".

### Where the time goes

The PRD target is a typical phrase in under 3 seconds. Steps 2, 3, 6 and 7 take about a millisecond or less together, so the LLM call in step 5 is almost the whole budget; network and TLS add roughly 100 to 300 ms. These figures are estimates, not measurements, and Week 3 logs the LLM call's duration so the real numbers replace them.

- **One LLM call per request, and no automatic retry.** A timeout usually means the provider is slow or down, so a retry would likely fail again and double the wait. It may also be billed again, which complicates the spend guard. Retries and a circuit breaker belong to V3.
- **A generous LLM timeout instead.** It is a config value, with a starting guess of 6 seconds, to be set in "Classifier design" once the real latencies are known. The server write timeout is set above it with margin, so the server never cuts off a call that is still allowed to finish.
- **The deadline comes from the request context.** If the visitor closes the tab, the context is cancelled and the LLM call stops, so we do not wait for, or pay more for, an answer nobody will see.
- **A failed request shows a "Try again" button.** The visitor decides whether to retry, and the retry goes through the spend guard like any other request.
- **The 3 second criterion describes the typical case.** The timeout is the worst-case limit. The PRD does not change.

### Decision: outcomes versus errors, a generous timeout and no retry

- **Decision:** `unclear` and a thin or empty bucket are outcomes: expected results with a friendly message. Timeout, provider error and invalid output are typed errors. The LLM call is made once per request with a generous, configurable timeout, derived from the request context, and a failed request offers a "Try again" button instead of retrying automatically. The PRD's 3 second criterion stays, as a typical-case target.
- **Rejected alternative:** an automatic retry after a timeout, with the PRD loosened to about 10 seconds; treating `unclear` as an error.
- **What breaks if wrong:** if `unclear` were an error, a normal answer would take the failure path, be logged as a fault and inflate error counts. If the timeout is too short, slow but healthy answers become errors. If it is too long, a stuck provider holds connections, and the visitor waits with no feedback. Without the context deadline, closed tabs still cost money. The cost of no retry is that a transient provider blip reaches the visitor as an error, which the "Try again" button softens.

### Decision: the spend guard runs after validation, just before the paid call

- **Decision:** the spend guard is not middleware. It runs after the handler has validated the input, directly before the LLM call, so a request rejected as bad input never counts against the budget. This supersedes the middleware order in the 2026-11-04 decision log row. The mechanism, the layers and the exact placement (around the `Matcher` or the `Classifier`) are settled in "Spend guard" (Q5).
- **Rejected alternative:** the guard in the middleware chain, before the handler.
- **What breaks if wrong:** a guard before validation counts requests that cost nothing, so a flood of empty submissions could use up the daily budget and lock out real visitors. A guard that sits after the LLM call guards nothing. The cost of the chosen placement is that the handler must handle one more typed failure (budget used up), and that a per-IP limit needs the client IP passed down from the HTTP layer.

## Routes and API contract

TBD

## Data model

### Decision: mood taxonomy (Q1, Q8)

- **Decision:** ten fixed moods, defined once in a single embedded JSON file (path settled with the Week 2 layout). Each mood has a `key` (lowercase ASCII slug, unique, stable, and never `unclear`), a `label` for display, and a one-line `definition`. Code contains no mood literals; a mood name appears only in the taxonomy file, the dataset tags, and the tests and eval. The loader checks every dataset tag against the taxonomy at startup, so a bad tag stops the app from booting. The classifier prompt and the enum validation are built from the taxonomy keys and definitions, and templates show the `label`. `unclear` is not a mood: it is a code constant for the classifier's no-mood output.

  | key | definition |
  | --- | --- |
  | `happy` | Content, cheerful or joyful about how things are right now. |
  | `sad` | Grief, sorrow or a low mood, with no blame or grudge. |
  | `angry` | Frustration, irritation or rage, often aimed at someone or something. |
  | `calm` | Quiet and tranquil; wanting to pause and absorb the moment. |
  | `anxious` | Tense and stressed about an uncertain future, often expecting the worst. |
  | `nostalgic` | A bittersweet longing for a past moment or time that still matters, whether the memory is happy or painful. |
  | `lost` | Unsure what you feel, numb or blank, disconnected from yourself. |
  | `love` | Romantic affection and longing: falling for someone, or basking in those feelings. A positive sentiment. |
  | `heartbreak` | The pain of love gone wrong: rejection, betrayal, unrequited love or a breakup. Hurt and loss, not rage. |
  | `energetic` | Fired up and ready to act; drive or anticipation, like a fresh start, a new project or a gym goal. |

  Love is romantic only in V1; affection for family or friends has no bucket. `heartbreak` is separate from `love` so that a heartbroken visitor gets results about the pain, not a mix with happy love songs.
- **Rejected alternative:** hardcoding mood names in each place that uses them; numeric ids instead of readable keys; one `love` bucket covering heartbreak; an LLM-returned tone score within a mood (a second dimension that is unreliable to produce, can't be checked by the eval, and doubles curation).
- **What breaks if wrong:** a mood name appears in about seven places (dataset tags, prompt, enum validation, catalog lookup, templates, eval and tests, docs). Renaming a key after curation means editing every tag for it, and without the startup check a typo'd tag silently drops an item from results with no error. If tests or the eval read their expected moods from the taxonomy file, a wrongly renamed or deleted key changes both sides and the tests still pass, so they hardcode the keys on purpose. Vague definitions blur the buckets, since the prompt is built from them.

### Decision: song link format (Q9)

- **Decision:** each song item carries `title`, `artist` and `urls`, a map keyed by service. The only allowed keys are `spotify` and `youtube`. Values are direct track URLs, not search URLs. At least one URL is required, not both. The loader validates at startup that the keys are in the allowed set, every URL is `https`, and the host matches its service (`open.spotify.com` for `spotify`; `youtube.com` or `youtu.be` for `youtube`). Links are link-out only: no embedded players and no music service API in V1. Every link is checked by hand before launch, alongside the mood-tag review.

  ```json
  { "type": "song", "mood": "nostalgic", "title": "...", "artist": "...",
    "urls": { "spotify": "https://open.spotify.com/track/...", "youtube": "https://www.youtube.com/watch?v=..." } }
  ```
- **Rejected alternative:** search URLs (never break, but land on a results page and may pick the wrong version); a single `url` field (no fallback, and no label for the service); a list of URLs (the template would have to guess the service from the URL); requiring both services (blocks curation when a track is missing on one, and doubles the lookup).
- **What breaks if wrong:** direct links rot, since tracks get removed or region-locked, so a dead link is a broken result with no error. The second service is the fallback, and the pre-launch hand check catches what is already dead. A typo in a service key would show an unlabeled or missing link, so the key set is validated like the mood taxonomy. A URL pasted into the wrong field would send the visitor to the wrong place, which the host check prevents. Storing `title` and `artist` leaves room for a music API to replace hand-copied links later (Q13).

The quote schema, the dataset file location and the full loader rules are TBD.

## Classifier design

TBD. The provider and model are decided in "Tech stack".

## Catalog and selection

TBD

## Input validation

TBD

## Spend guard

TBD. This topic also answers Q5.

### Decision: spend guard mechanism (Q5)

- **Decision:** TBD
- **Rejected alternative:** TBD
- **What breaks if wrong:** TBD

## Failure modes

TBD

## Security and privacy

TBD

## Observability

TBD

## Testing strategy

TBD

## Config and deployment

### Decision: hosting target (Q15)

- **Decision:** the host is not chosen in Weeks 1-5. Build and test come first, and the host is picked at the start of Week 6. Until then the service stays portable by meeting these requirements, which any candidate host (Fly.io, Render, Railway, Cloud Run) supports:
  - It ships as one container image built from a multi-stage Dockerfile, run as a non-root user.
  - All configuration and secrets come from environment variables; none are baked into the image.
  - It listens on the port the platform provides.
  - It handles the platform's stop signal with graceful shutdown, and exposes `/healthz`.
  - It runs as a single instance. Any in-memory counter used by the spend guard (Q5) is a soft layer that resets on restart, so the provider's own spend cap is the durable one.
  - If a per-IP limit is used, the client IP is read only from the header the chosen platform sets.
- **Rejected alternative:** choosing the host now (it blocks nothing in the build, and the research is still in progress); deploying early to a throwaway host (adds work before there is anything to test).
- **What breaks if wrong:** a host with idle spin-down would make the first visitor wait about a minute, which breaks the under-3-second success criterion. A host that scales to zero resets in-memory spend counters. A requirement quietly dropped during the build, such as a hardcoded port or a baked-in secret, turns the Week 6 move into a refactor.

The environment variable list and the deploy and rollback steps are TBD.

## Repo layout

TBD. The V1 to V4 plan is in [repo_structure.md](../repo_structure.md) (draft); this section will record the V1 part and link there.

## Work breakdown

TBD

## Risks and what V2 inherits

TBD

## Frontend and design system

### Decision: server-rendered pages

- **Decision:** pages are rendered on the server with Go `html/template`. htmx handles interactivity (a loading state and result swaps in V1, streamed results over Server-Sent Events in V3), with small vanilla JS where needed. Templates and static assets are embedded in the binary with `go:embed`. htmx is vendored at a pinned version in `web/static/` and never loaded from a CDN. A JSON API (`POST /api/match`) stays alongside the pages. Match logic returns data, and the page handler and the JSON handler only render it. A separate single-page app is a V4 revisit (Q16).
- **Rejected alternative:** a single-page app (React, Svelte) from V1; a separate frontend project; loading htmx from a CDN.
- **What breaks if wrong:** if V4 needs an interaction that CSS and htmx do not handle well, or a second client such as a mobile app, the frontend splits out (Q16). That stays a frontend-only change as long as handlers never build HTML inside the match logic. `html/template` escapes by default, so `template.HTML` is never used on a phrase or any LLM output. Vendored htmx must be upgraded by hand. V3 streaming needs the server's write timeout and the host's proxy buffering checked, because Week 4's server timeouts could cut a long-lived stream.

### Decision: design pass

- **Decision:** V1 includes one timeboxed design pass of about 1.5 hours, in Week 4, so the site looks clean and contemporary. The look is defined once, as design tokens in CSS variables in one file, and every page and later category reuses them. Week 1's throwaway page uses a minimal version of the tokens so they are tested early. What goes into the tokens, the components and the page states is decided in the rest of this section, not here. If Week 4 runs long, polish items are cut first.
- **Rejected alternative:** styling page by page during feature work; an unbounded polish phase; leaving the look to V4.
- **What breaks if wrong:** without one tokens file, each page and each V3 category drifts into its own look, and a restyle means editing every template. Without the timebox, polish takes hours from the backend work this project exists to teach.

The token list, components, page states and accessibility basics are TBD.
