# Moodwise

Mood-to-media matcher: the user types a phrase (a feeling, lyric, word) and gets back songs, quotes, and later books and movies that match the mood. A personal project built in Go, in versions V1 to V4, at roughly 3-4 hours a week.

**Current version: V1.** Go, fixed mood buckets classified by an LLM API, songs and quotes only, hand-curated flat JSON dataset, no embeddings, no database.

No code exists yet, so there are no build or test commands. Add them here with the first code PR.

## Project context

These load every session:

@docs/decision_logs.md
@docs/v1/weekly_plan.md
@docs/v1/tdd.md

Read these when relevant:

- docs/project_plan.md: versions, and what each one deliberately skips. Use it for scope checks.
- docs/open_questions.md: parked items. Check it when a week or milestone starts, and raise anything whose revisit point has arrived.
- docs/v1/prd.md: V1 requirements.
- docs/v1/retro.md: written in the last week of the version. It hands over what the next version needs.
- docs/todo.md: chores and follow-ups, each with a trigger. Check it at session start and at milestone wrap-up, and raise anything whose trigger has arrived.

The PRD and TDD are the spec. Follow them, and flag any deviation as a plan change or a decision.

The project plan is directional: it says what each version aims for and skips. The PRD is exact: it gives the real numbers and behavior. A PRD that is more specific than the plan is expected, not a conflict. Only treat them as conflicting when they contradict each other.

When a version ends: finish its retro, then create `docs/v<N>/` for the next one, and update the current version line and the imports above.

## Working style

The user is learning backend engineering through this project, so understanding matters more than speed.

- After generating code, briefly explain the key design decisions, then ask one question that checks the user understood the "why", not just the "what".
- Do not move on to the next piece until the user can explain the current one. If the answer is shaky, explain again and re-check.
- Apply production-grade practice wherever design or code quality matters (error handling, testing, observability, security, simplicity), and say briefly why.
- If the user makes an inappropriate choice, push back clearly, explain the reasoning, and give the better alternative. Do not go along with it.

## Project hygiene

Update docs in the same PR as the change that caused the update. A PR that makes a decision without logging it is incomplete. Not every PR has a decision, an open question or a to-do, so the PR template asks for each as "logged, or none".

| Trigger | Action |
| --- | --- |
| Decision: a choice between alternatives, or "let's go with X" | Append a row to docs/decision_logs.md |
| Open question: "later", "not now", an unresolved doubt | Add a row to docs/open_questions.md with the next free Q number, the stage it blocks (when to revisit it) and its status |
| Plan change: the scope, order or timing of a milestone or week changes | Edit the project plan or weekly plan, and show before and after in the PR description |
| Scope creep: an idea that belongs to a later version | Say so, check it against the current version's "deliberately skips" list, and park it as an open question tagged with its target version |
| Contradiction: a new decision conflicts with an earlier decision or the plan | Name the conflict before continuing, and propose which file should change |
| Chore: a follow-up that is not a decision or an open question (for example "add X when Y lands") | Add a row to docs/todo.md with a trigger for when to do it |

Always keep all docs consistent with each other: no conflicts and nothing stale. When a change touches one doc, check the README, project plan, weekly plan, PRD, TDD, decision log and open questions for statements it affects, and fix them in the same PR.

The decision log is append-only. A reversed decision gets a new row and the old row stays. A resolved open question is removed from docs/open_questions.md and logged as a decision row ending with "(Resolves Q#.)". Question IDs are never reused.

## Code conventions

- Never leave TODO or FIXME comments in code, and don't write paragraph-long "to do later" comments. Add the follow-up to docs/todo.md with a trigger instead.
- Comments explain why the code is the way it is, not what is still missing.

## Session rhythm

- At the start of each session, ask whether the docs are in sync with the latest planning discussion in the Claude Project.
- When the user is wrapping up or finishing a milestone, give one consolidated list of pending doc updates, plus any open questions older than the current milestone.
