# moodwise

Mood to Media matcher for every text you type.

Type a phrase (a feeling, a lyric, a single word) and Moodwise returns songs and quotes that match its mood. Books and movies come in later versions.

## Status

**Early development, on V1.** There is no runnable code yet; the repo currently holds the project docs.

Moodwise is a personal project for learning backend engineering in Go. It is built in versions, V1 to V4, at roughly 3-4 hours a week, so understanding the design takes priority over shipping fast.

## V1 scope

| Area | V1 approach |
| --- | --- |
| Language | Go |
| Mood detection | An LLM API classifies the phrase into one of 6-8 fixed moods, or "unclear" if it describes no feeling |
| Media | Songs (with links out) and quotes |
| Data | Hand-curated flat JSON dataset, 50-100 items |
| Cost | A basic spend guard on the public endpoint |
| Deliberately skipped | Embeddings, a database, books and pictures, full rate limiting |

See the [project plan](docs/project_plan.md) for how the versions build on each other.

## Docs

| Path | What it holds |
| --- | --- |
| [docs/project_plan.md](docs/project_plan.md) | Overall plan across versions |
| [docs/decision_logs.md](docs/decision_logs.md) | Decisions made, and why |
| [docs/open_questions.md](docs/open_questions.md) | Items parked for later |
| [docs/todo.md](docs/todo.md) | Chores and follow-ups, each with a trigger |
| [.claude/skills/](.claude/skills/) | Project skills for Claude Code, such as `/create-pr` |
| [docs/v1/](docs/v1/) | Weekly plan, PRD, TDD and retro for V1 |

Each milestone gets its own `docs/v<N>/` folder. [CLAUDE.md](CLAUDE.md) is the working guide for Claude Code on this repo.
