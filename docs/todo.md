# To-do

Chores and follow-ups that are not project decisions or open questions. Each item has a trigger, so it comes up when the time is right. Do not leave TODO or FIXME comments in code: add a row here instead.

When an item is done, delete the row. Anything that is a real choice between alternatives belongs in [decision_logs.md](decision_logs.md), and an unresolved doubt belongs in [open_questions.md](open_questions.md).

| Added | Item | Do when |
| --- | --- | --- |
| 2026-11-01 | Add build, test and run commands to CLAUDE.md (for example `make test`, `make vet`, `make run`) | The first code PR lands the Makefile (Week 2) |
| 2026-11-01 | Update the Claude Project instructions: the repo docs are now the source of truth, and the Project plan is directional while the PRD is exact | Once the docs PR is merged |
| 2026-11-01 | Migrate any remaining Project context copies into the repo, then treat the Project copies as periodic snapshots | The first code PR lands |
| 2026-11-01 | Add a `/resolve-question Q#` project skill: removes the row from open_questions.md, adds the "(Resolves Q#.)" decision row, and fills the TDD entry. It must check that the answer is the user's own and not draft it. List it in CLAUDE.md and log a decision row in the same PR | Q1 has been resolved by hand, so the real steps are known |
| 2026-11-01 | Add a `/wrap-up` project skill: reports pending doc updates, open questions older than the current milestone, and to-dos whose trigger has arrived. List it in CLAUDE.md and log a decision row in the same PR | The end of Week 1, or the first time a wrap-up list is written by hand |
| 2026-11-02 | Decide the classifier prompt's tie-breaker for phrases that fit two moods (for example "choose the mood the feeling is mostly about"), and cover heartbreak versus angry/sad and love versus nostalgic in the eval's ambiguous set | Week 3, when the classifier prompt is written |
| 2026-11-02 | Add a link checker that flags dead song URLs in the dataset | Before launch the links are checked by hand; build the checker when the dataset is more than a few weeks old or a dead link is found |
| 2026-11-04 | Confirm the Gemini Flash-Lite model ID, price and Go SDK shape against Google's own docs (the TDD figures came from third-party aggregators), and re-check that the OpenAI fallback model still exists | Week 3, before the LLM classifier is built |
| 2026-11-04 | Install Go 1.27 and `make` on the dev machine (neither is on the PATH today) and check `go version` | Before the first code PR, at the start of Week 2 |
| 2026-11-04 | Revisit adding `staticcheck` or golangci-lint to CI | When the codebase passes a few thousand lines, or a bug appears that a linter would have caught |
| 2026-11-04 | Choose the CSS approach (hand-written CSS variables, or Tailwind via its standalone CLI) and log a decision row | Before the Week 1 throwaway page is styled |
| 2026-11-04 | Vendor htmx at a pinned version into `web/static/` and record the version | The first page that uses it |
| 2026-11-04 | Choose vector storage and cache, and log decision rows. Likely: precomputed embeddings with in-memory cosine similarity, and an in-process TTL cache behind an interface | V2 planning |
| 2026-11-04 | Choose the Postgres access and migration tools (likely `pgx`, `sqlc`, `goose`, `pgvector`), the metrics library and the retry and circuit-breaker approach, and log decision rows. Also recheck the server write timeout against streamed (SSE) responses | V3 planning |
| 2026-11-04 | Choose the auth approach (likely cookie sessions in Postgres) and the rate limiter (likely `x/time/rate`), and adopt `chi` only if per-route middleware gets messy. Log decision rows | V4 planning |
