# V1 TDD outline

A working checklist for drafting [tdd.md](tdd.md). It lists every section the TDD needs, what we already know about it, what is still to decide, and where each topic stands. Update the Status column when a topic's TDD entry lands. Delete this file once the TDD is final.

The goal of the TDD: detailed enough (tech stack, system design, design choices and their justification, implementation design, no code) that a developer can plan the work from it.

The TDD is organised by these topics, using the same names as headings and no numbers. The numbers below are only for quick reference in this file. Each topic holds its own design and its decisions (decision, rejected alternative, what breaks if wrong), and each decision also gets a row in [decision_logs.md](../decision_logs.md). There is no separate "Key decisions" section.

## Sections

| # | Section | What we already know | What we still need to decide | Status |
|---|---|---|---|---|
| 1 | **Overview, goals, non-goals** | PRD: problem, audience, scope, success criteria, what V1 skips | Nothing. Short summary that links to the PRD | Drafted, awaiting your review |
| 2 | **Tech stack** | Go, an LLM API for classification, a flat JSON dataset, no database, no embeddings, server-rendered `html/template`, `slog`, config via env vars | ~~LLM provider and model~~ (done: Gemini Flash-Lite, OpenAI fallback), ~~hosting target~~ (deferred to Week 6, Q15; portability requirements are in the TDD), ~~Go version, router, test and lint tools~~ (done: Go 1.27, stdlib `ServeMux`, stdlib `testing`, `go vet` and `gofmt`), ~~frontend approach and long-term stack~~ (done: modular monolith, server-rendered with htmx, JSON API alongside, V3 and V4 libraries chosen at their versions) | Done |
| 3 | **Architecture and components** | Components named in the plan: handler, `Classifier`, `Catalog`, dataset loader, taxonomy file, templates | Component boundaries and dependency direction; the V2-V4 extension seams (what each later version swaps or wraps, per the project plan) shown on the diagrams; Q2: why `Classifier` is an interface (draw the handler both ways). Mermaid diagrams (high level and close-ups) live in `docs/v1/design/` and are linked here | Done: every component has been reviewed, and the decisions on `Matcher`, `Classifier` (Q2 resolved), taxonomy, `Catalog`, templates, middleware, and config with the composition root are made. The outcome and error types are left to request flow, failure modes and classifier design |
| 4 | **Request flow** | Happy path and `unclear` path from the PRD | ~~Step-by-step flow, where time goes (3 s budget), where each step can fail~~ (done). Still to set: the LLM timeout number (Classifier design, after Week 3 measurements) | Done, awaiting your review |
| 5 | **Routes and API contract** | `POST /api/match`, `/healthz`, consistent JSON error shape | Decided: two entry points (`POST /match` for HTML, `POST /api/match` for JSON), phrase by POST only, real status codes on both with htmx swapping error responses, JSON shapes, error view model. Still to settle in Failure modes: the final error codes, messages and which kinds offer Try again; quote fields (Data model) | Done, awaiting your review |
| 6 | **Data model** | Taxonomy file: key, label, definition (done). Song items: `title`, `artist`, `urls` map (done) | ~~Quote schema, dataset file location and format, full loader validation rules, dedup rules, an ID per item~~ (done: embedded `content/items.json`, quote `text` and `author`, explicit ids, one mood per item, a loader that reports every error, thin-bucket warnings) | Done, awaiting your review |
| 7 | **Classifier design** | Output is a mood key or `unclear`. Prompt built from the taxonomy. Output validated against the enum. Timeout via `context`. Key from env, never logged | Prompt structure, structured-output approach, timeout value, retry or no retry, tie-breaker for two-mood phrases (see todo.md), fallback on invalid output | Not started |
| 8 | **Catalog and selection** | Random sample, up to 3 songs and 2 quotes, injected random source, thin buckets show fewer items | ~~Whether the dataset is indexed by mood at startup, how a bucket with 0 songs but some quotes behaves~~ (done: immutable index, non-mutating sample, injected random function, empty bucket is a result, unknown mood is an error) | Done, awaiting your review |
| 9 | **Input validation** | Length cap, empty and whitespace input rejected | The cap number, unicode and control characters, what the friendly message says | Not started |
| 10 | **Spend guard** | Required by the PRD. Candidates: input cap, per-IP limit, global daily counter, provider spend cap (Q5) | Which layers, with what numbers, and what the visitor sees when the guard trips | Not started |
| 11 | **Failure modes** | Five modes from the plan: LLM timeout, invalid LLM output, empty bucket, bad input, `unclear` or injection | For each: visitor message, status code, log line, who gets alerted | Not started |
| 12 | **Security and privacy** | Prompt-injection stance and the eval's injection set. Phrases not stored but sent to a third party | Secret handling, what is logged about a phrase, HTML escaping, security headers, outbound link attributes | Not started |
| 13 | **Observability** | `slog`, `/healthz`, provider dashboard | Log fields, request ID, which events are logged, what `/healthz` checks | Not started |
| 14 | **Testing strategy** | CI runs `go vet` and unit tests with fakes; `-race` from Week 5. Eval on demand: 90% typical, 100% injection. Hand review of tags and links | What each layer is for, e2e setup with a fake LLM server, eval harness shape, test-data approach | Not started |
| 15 | **Config and deployment** | Multi-stage Dockerfile, non-root, host options (Fly.io, Render, Railway), healthcheck, graceful shutdown | Env var list, deploy and rollback steps. Host choice is deferred to Week 6 (Q15); portability requirements are already in the TDD | Partly done (hosting portability) |
| 16 | **Repo layout** | `cmd/` and `internal/` | Package list and which package may import which. The layout must ramp up to V2-V4: organize by component, not by version, and build seams, not empty scaffolding. The V1-to-V4 plan is in [repo_structure.md](../repo_structure.md) (draft); this section records the V1 part and links there | Not started |
| 17 | **Work breakdown** | The weekly plan | Tasks per week with dependencies and "done" criteria, so a developer can plan from it. Written last; closes Q10 | Not started |
| 18 | **Risks and what V2 inherits** | Risks section in the weekly plan; the retro comes later | What stays stable so V2 can swap the matcher (Q11, Q12 and Q13 are the V2 hooks) | Not started |
| 19 | **Frontend and design system** | Server-rendered `html/template` with htmx (vendored, pinned), JSON API alongside, assets embedded, design pass in Week 4, tokens in one CSS file (decided; see the decisions in TDD section 19) | CSS approach (todo.md); token list and palette; component list; page and state inventory (home, results, `unclear`, error, thin or empty section); where htmx is used in V1; accessibility basics (contrast, focus, reduced motion); how the V3 SSE stream attaches. Per-mood accent colours are parked in nice_to_have_features.md | Partly done (server-rendered approach, design pass) |

## Order of work

Each topic depends on the ones before it:

1. Tech stack (2), done
2. Architecture and request flow (3, 4); Q2 is answered here
3. Data model (6) and catalog (8); the quote schema blocks the dataset work
4. Classifier (7)
5. API, validation and failure modes (5, 9, 11), then the frontend and design system (19)
6. Spend guard (10), then security (12) and observability (13)
7. Testing (14) and deployment (15)
8. Repo layout (16), moved here on 2026-11-08 so the layout is organised around what the implementation topics decided; it must be final before Week 2's repo setup
9. Work breakdown (17), then risks (18)

For each topic: discuss, state the decision in your own words, write the TDD entry, add the Decision Log row, update the Status column, then move on.

## Open items that touch the TDD

- Q5: spend-guard mechanism (topic 10)
- Q10: do the Week 2-4 time estimates still hold, including the design pass (topic 17)
- Q16: server-rendered or a separate frontend, a V4 revisit (topic 19)
- Classifier tie-breaker for two-mood phrases, in [todo.md](../todo.md) (topic 7)
- Q15: hosting target, deferred to Week 6 (topic 15 keeps only the host-neutral parts)
- Quote schema and dataset path, needed before curation resumes (topic 6)
