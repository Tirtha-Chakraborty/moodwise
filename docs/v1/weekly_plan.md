# Moodwise V1: Week-by-Week Plan

Last updated: 2026-11-04

## Overview

- **Duration:** 6 weeks, about 3-4 hrs/week (roughly 20-22 hrs total)
- **Language:** Go (see [Decision Log](../decision_logs.md))
- **Approach:** Backend-first, one thin end-to-end slice per week (see the Decision Log, Q3)
- **Rule for slices:** each week's demo must be the whole loop, thin, not one layer thick. "The DB layer is done" is a horizontal layer and does not count as a demo.
- **Week 1 is the scope and docs week.** There is no Week 0 (see the Decision Log, Q4).

## Working agreement

- After each piece of generated code: a brief explanation of the key design decisions, plus one question that checks understanding of the *why*.
- No moving to the next piece until the current one can be explained in your own words.
- Production-grade practice wherever design or code quality matters. Inappropriate choices get pushed back on and corrected.
- Each week ends with an **exit gate**: explain the week's key decision back before starting the next week.
- A week is not closed until every Open Question that blocks it is resolved.

## At a glance

| Week | Demoable slice at the end | Main focus |
|---|---|---|
| 1 | Static page showing the frozen taxonomy and ~20 dataset items, plus PRD and TDD draft | Scope, docs, taxonomy freeze, dataset schema |
| 2 | **Walking skeleton:** form, handler, `FakeClassifier`, catalog, results page with mood label and song links, running locally | Repo setup, interfaces, loader with validation, `/healthz`, CI, spend-guard design (Q5) |
| 3 | **Real classification:** a phrase hits the LLM and returns mood-matched items | LLM classifier, enum validation, timeouts, unit tests with a fake client |
| 4 | **Hardened and reviewed:** bad input, empty buckets, LLM failures show sensible errors; spend guard in place | Handler validation, error shape, spend guard (per Q5), design pass, review pass |
| 5 | **Trustworthy:** test suite green with `-race`, classifier eval (90% typical, injection set 100%), injection cases checked | Handler and e2e tests, eval set, graceful shutdown |
| 6 | **Live at a public URL** | Dockerfile, deploy, smoke test, README, retro |

## Week 1: Scope and docs (~3.5 hrs)

**Order matters.** Curating dataset items is the first work that bakes bucket names into the project, so the taxonomy question must be answered first.

1. Draft the **PRD** (1 page): problem, user flow, V1 in/out of scope (from the Project Plan), success criteria. Example: "a phrase returns 3-5 relevant items in under 3s, deployed at a public URL."
2. Draft the TDD topic by topic, starting with the taxonomy in the data model topic (answers Q1). Each topic holds its design and its decisions: the decision, the rejected alternative, and what breaks if it's wrong.
3. **Freeze the mood taxonomy:** 6-10 buckets (including a "lost" bucket for uncertain feelings), one-line definition each. The classifier's separate "unclear" output is not a bucket.
4. Draft the rest of the **TDD (first draft)**: components, request flow, API contract (`POST /api/match`), JSON item schema, failure-modes table (LLM timeout, invalid LLM output, empty bucket, bad input, unclear / injection input).
5. Write the dataset schema (type: song or quote; for songs, title, artist and a map of Spotify and YouTube URLs) and curate the first ~20 items.
6. Build the throwaway static page that renders the taxonomy and sample items. It uses a minimal `tokens.css` (palette, type scale, spacing), so the design tokens are tested from the start.

**Blocking questions:** none left (Q1, Q8 and Q9 were resolved on 2026-11-02; see the Decision Log. Q1 was: what breaks, and in how many places, if a bucket is renamed or added after the dataset is half built).
**Exit gate:** explain why the taxonomy had to be frozen before building anything else.

## Week 2: Design and walking skeleton (~3.5 hrs)

- Finalize the TDD and log each decision in the Decision Log (taxonomy, LLM provider and model, backend-first slicing). The hosting target is deferred to Week 6 (Q15).
- Decide the spend-guard mechanism in the TDD.
- Repo setup: `cmd/` and `internal/` layout, config via env vars, `slog` structured logging, `/healthz`, Makefile, basic CI (`go vet`, `go test`; fakes only, no real API calls).
- Define the interfaces: `Classifier` (phrase to mood) and `Catalog` (mood to items).
- Implement `FakeClassifier` (always returns one mood) and the dataset loader with **startup validation**: every item's mood is in the taxonomy, no duplicates, no empty fields. The loader also validates type and song link presence.
- Bare-bones form and results page (server-rendered `html/template`) wired end to end. The results page shows the detected mood, two sections, song links and the feedback form link.
- Curate another ~25 items.

**Blocking questions:** Q5, Q10. Q2 (why `Classifier` is an interface) was resolved on 2026-11-04; see the Decision Log and the TDD's "Architecture and components" section.
**Exit gate:** explain why `Classifier` is an interface and not a direct SDK call in the handler.

## Week 3: Real classification (~4 hrs)

- Implement the LLM classifier. It returns a bucket or `unclear`:
  - request timeout via `context`
  - prompt constrains output to the taxonomy, ideally structured JSON, and says to return `unclear` for non-feelings and instructions
  - **validate the response against the enum**, with a safe fallback or clear error if the model returns something off-list
  - API key from the environment, never logged
- Swap it in for `FakeClassifier` without touching the handler or templates.
- Implement the real catalog filter. It picks a random sample with an injected random source, and a thin bucket shows fewer items.
- Table-driven unit tests for the filter and for response parsing, using a fake client (no real API calls).
- Finish curating the dataset (50+ items, roughly balanced across buckets).

**Exit gate:** explain why LLM output is untrusted input and what your validation protects against.

## Week 4: HTTP hardening, frontend polish, review (~3.5 hrs)

- Handler: input validation (length cap, empty or whitespace input), consistent JSON error shape, correct status codes, server timeouts (`ReadHeaderTimeout` and others). `unclear` shows the friendly message.
- **Design pass** (timebox about 1.5 hrs, see the TDD's "Frontend and design system" section): design tokens and reusable components, with the specifics set there. If the week runs long, cut polish items first. The feedback form link goes on both pages.
- **Cost and abuse guard:** implement the spend guard chosen in Q5. Candidate layers: input length cap, simple per-IP limit, global daily counter, hard spend cap on the provider dashboard. V1 skips scale, but a public endpoint that spends money per request needs a basic safeguard. Full rate limiting stays in V4.
- Review pass: read your own diff first, then walk it through against a checklist (error handling, secrets, logging, input validation).

**Exit gate:** explain why the spend guard belongs in V1 even though rate limiting is a V4 feature.

## Week 5: Testing and hardening (~3.5 hrs)

- Handler tests with `httptest`: happy path, bad input, classifier failure, empty bucket.
- End-to-end test with a fake LLM server, plus `go test -race`.
- Classifier eval (on demand, real classifier, not in CI): ~20 typical phrases at 90%, plus a separate injection set of 5-8 phrases at 100% (result must be "unclear"), such as "ignore previous instructions." Write acceptable moods before running. Also review every dataset item's mood tag by hand.
- Fix what the review and tests turn up. Verify graceful shutdown and that logs hold no secrets or full user input you wouldn't want stored.

**Exit gate:** explain what each test layer catches that the others don't.

## Week 6: Ship (~3 hrs)

- Multi-stage Dockerfile (small image, non-root user).
- Choose the host first (Q15), then deploy to it (candidates: Fly.io, Render, Railway, Cloud Run). Secrets via the platform, plus a healthcheck.
- Smoke test the live URL, then trigger one failure on purpose (for example a bad API key) and confirm it fails gracefully.
- README: what it is, how to run it, architecture sketch, known limitations. Limitations must say it is not injection-proof, that phrases go to a third-party AI provider, and that the app doesn't store them.
- Update the Decision Log and write the short retro in [retro.md](retro.md) listing what V2 needs from V1's design.

**Exit gate:** walk through a request from browser to response, naming where it can fail and what the user sees in each case.

## Risks to watch

- **Dataset curation** is the hidden time sink, so it is spread across Weeks 1-3. Don't leave it to the end.
- **Curation load:** 10 moods x ~5-10 items. If it runs long, trim a mood rather than thinning all of them.
- **Week 4 load:** the design pass adds about 1.5 hrs to a week that already holds the spend guard, validation and the review pass. Q10 checks the estimate. If it does not fit, cut polish items first, then move the pass to Week 5.
- **Taxonomy churn:** changing buckets mid-build invalidates the prompt, the tags, and the tests.
- **Scope creep:** no embeddings, no third category, no database. Those are V2 and V3.
- **Docs ballooning in Week 1:** keep the PRD to one page and treat the TDD as a first draft to revise in Week 2.
