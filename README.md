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
| Mood detection | An LLM API classifies the phrase into fixed mood buckets |
| Media | Songs and quotes |
| Data | Hand-curated flat JSON dataset |
| Deliberately skipped | Embeddings and a database |

See the [project plan](docs/project_plan.md) for how the versions build on each other.

## Docs

| Path | What it holds |
| --- | --- |
| [docs/project_plan.md](docs/project_plan.md) | Overall plan across versions |
| [docs/decision_logs.md](docs/decision_logs.md) | Decisions made, and why |
| [docs/open_questions.md](docs/open_questions.md) | Items parked for later |
| [docs/v1/](docs/v1/) | Weekly plan, PRD and TDD for V1 |

Each milestone gets its own `docs/v<N>/` folder. [CLAUDE.md](CLAUDE.md) is the working guide for Claude Code on this repo.
