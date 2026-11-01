# Decision log

Decisions made on this project, and why. This log is append-only: a reversed decision gets a new row and the old row stays.

| Date | Decision | Rationale | Alternatives rejected |
| --- | --- | --- | --- |
| 2026-11-01 | Keep project docs in the repo under `docs/`, with a `docs/v<N>/` folder for each milestone | Docs sit next to the code and change in the same PR, so the reason for a change stays attached to it | Keeping docs only in the Claude Project, where they drift from the repo |
| 2026-11-01 | Name the shared docs `project_plan.md`, `open_questions.md` and `decision_logs.md` | Matches the layout the user specified | `decisions.md` and `open-questions.md`, the earlier example names |
| 2026-11-01 | `CLAUDE.md` imports only the decision log and the current version's weekly plan and TDD, and lists other docs by path | Keeps per-session context small while always loading what constrains current work | Importing every doc in every session |
