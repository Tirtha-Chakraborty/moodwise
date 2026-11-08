# Moodwise: Repo Structure Plan

**Status:** Draft (2026-11-04). The architecture decision (the handler depends on a `Matcher`) is made and confirms the `match/` package below. The layout is final once the request flow is written. Once confirmed, log it in [decision_logs.md](decision_logs.md). The V1 TDD's repo layout section records only the V1 part and links here.

This repo is reused for V1 to V4. The goal is that later versions add or replace packages, and never force a rewrite of what already exists. The V3 and V4 parts are a forecast from the [project plan](project_plan.md), and each version's retro revisits them.

## Final shape (end of V4)

Tags show the version that adds each entry.

```
moodwise/
├── cmd/
│   ├── moodwise/          [V1] server entry point: load config, wire components, run, shut down
│   ├── eval/              [V1] on-demand eval against the real classifier; [V2] adds matching-quality checks
│   └── embed/             [V2] offline tool that computes vectors for the curated items
├── internal/
│   ├── config/            [V1] env-var config, validated at startup
│   ├── domain/            [V1] core types (Mood, Item, Result, Section) and error kinds; imports nothing
│   ├── match/             [V1] Matcher interface: phrase in, results out
│   │   ├── bucket/        [V1] classify the phrase, then look up the mood bucket
│   │   └── semantic/      [V2] embedding similarity
│   ├── classifier/        [V1] interface, prompt building, enum validation
│   │   ├── gemini/        [V1] the only code that imports an LLM SDK
│   │   ├── openai/        [on pivot] fallback provider
│   │   └── fake/          [V1] test double
│   ├── catalog/           [V1] loader, startup validation, random selection from curated data
│   ├── source/            [V2] Source interface; catalog moves in as source/static; live API clients (movies, books)
│   ├── cache/             [V2] wrappers around a Matcher or Source
│   ├── resilience/        [V3] retries, backoff and circuit-breaker wrappers
│   ├── store/             [V3] Postgres repositories
│   ├── metrics/           [V3] latency per source, cache hit rate
│   ├── auth/              [V4]
│   ├── personalize/       [V4]
│   └── server/            [V1] routes, handlers, template wiring
│       └── middleware/    [V1] request id, logging, spend guard; [V4] rate limit, auth
├── content/               [V1] taxonomy.json, items.json, embed.go (hand-curated files only)
├── web/                   [V1] templates/, static/ (CSS tokens, vendored htmx, JS), embed.go; [V4] may become a separate frontend/ app (Q16)
├── e2e/                   [V1] black-box tests against a fake LLM server; [V3] fake sources and database
├── db/migrations/         [V3]
├── deploy/                [V3] local Postgres compose file (the Dockerfile stays at the root)
├── docs/                  [V1] versioned: docs/v<N>/, with design/ for Mermaid diagrams
├── .github/workflows/     [V1] CI
└── Dockerfile, Makefile, go.mod, README.md, CLAUDE.md
```

## What V1 builds, and what it leaves out

**Built in V1:** `cmd/moodwise`, `cmd/eval`, `internal/{config, domain, classifier (gemini, fake), catalog, server (with middleware), match}`, `content/`, `web/`, `e2e/`, `docs/v1/design/`, the Dockerfile, Makefile and CI workflow.

**Not built in V1 (no empty folders):** `embed/`, `semantic/`, `source/`, `cache/`, `resilience/`, `store/`, `metrics/`, `auth/`, `personalize/`, `db/`, `deploy/`, `openai/`. Each appears in the version that first needs it.

## `web/` layout (V1)

```
web/
├── templates/     layouts, pages and partials (song card, quote card)
├── static/
│   ├── css/       tokens.css (design tokens), then component and page styles
│   ├── js/        htmx (vendored, pinned version) and small scripts
│   └── img/
└── embed.go       go:embed, so one binary carries the whole frontend
```

## Rules that keep the structure clean

1. **Organize by component, never by version.** Docs are versioned; code is not. There is no `v1/` folder in `internal/`. Releases are marked with git tags such as `v1.0.0`.
2. **Dependency direction.** `cmd` depends on `server`; `server` depends on interfaces such as `match`; implementations depend on `domain`; `domain` imports nothing. Only `classifier/<provider>` imports an LLM SDK. Only `store` (V3) imports a database driver. `server` imports no provider code.
3. **A new package needs a reason:** embedding, an import boundary, independent replacement, or separate ownership.
4. **Interfaces are small and defined where they are used.** One is introduced when there is a second implementation or a test double to swap in.
5. **Cache, retries and the circuit breaker are wrappers** that implement the same interface as what they wrap, so adding them never changes callers.
6. **Curated content is separate from code.** `content/` holds only hand-curated source files. Generated files (V2 vectors) go in a separate place, decided in V2.
7. **One Go module, one repo.**
8. **Keep the eval independent.** Eval phrases and expected moods hardcode the mood keys and do not read `content/`.
9. **Handlers render, they do not decide.** Match logic returns data; the page handler and the JSON handler only render it, so another frontend can reuse the backend.

## Anticipated mechanical changes

These are the moves and renames already known. Each is a small, mechanical change and not a redesign.

| Version | Change | Size |
| --- | --- | --- |
| V2 | Add `match/semantic`, `cmd/embed`, `cache/`. The handler is untouched if it depends on `Matcher` | Add only |
| V2 | Introduce `source/` when the first live API arrives; `git mv catalog source/static` and update imports | Mechanical move |
| V2 | `classifier/` may shrink to supplying the displayed mood label, or be retired | Decided in V2 |
| V3 | Add `resilience/`, `store/`, `metrics/`, `db/migrations/`, `deploy/`; orchestrate sources in parallel; add a streaming (SSE) handler to `server` and recheck its write timeout | Add only |
| V4 | Add `auth/`, `personalize/`, rate-limit and auth middleware; decide whether `web/` stays or splits into `frontend/` (Q16) | Add, plus one decision |

## Open points

- **Package names:** `domain`, `match` and `server` are working names.
- **Import-boundary check in CI** is a candidate once there are enough packages to break the rules by accident.
