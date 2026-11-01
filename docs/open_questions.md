# Moodwise V1: Open Questions Log

Last updated: 2026-11-01

## Open

| # | Question | Raised | Stage it blocks | Status |
|---|----------|--------|-----------------|--------|
| Q1 | What breaks, and in how many places, if a mood bucket is renamed or added after the dataset is half built? (Answer by listing every place a bucket name appears as a string.) | 2026-10-03, Week 1 planning | Week 1: must be answered before curating dataset items and freezing the taxonomy | Open, awaiting your draft |
| Q2 | Why is `Classifier` an interface, and what would change in the handler if it called the LLM SDK directly? | 2026-10-03, Week 1 planning | Week 2: must be answered before the walking skeleton | Open, to be answered in the TDD "Key decisions" section |
| Q5 | Which mechanism(s) implement the V1 spend guard (provider spend cap, input length cap, global daily counter, per-IP limit)? The PRD states only the requirement. | 2026-11-01, PRD drafting | Week 2: close in the TDD before the build starts (built in Week 4) | Open. Candidate layers discussed, nothing chosen. On resolution, update the PRD's cost section and add a Decisions Log row. |
| Q8 | What are the final 6-8 moods and their one-line definitions? Candidate: happy, sad, nostalgic, calm, angry, hopeful, anxious, lost. | 2026-11-01, PRD drafting | Week 1: before dataset curation and the taxonomy freeze (also needs Q1) | Open, candidate proposed |
| Q9 | Which link format for songs: direct track URL (can break) or a search URL (more stable)? Which service? | 2026-11-01, PRD drafting | Week 1: dataset schema | Open |
| Q10 | Do the Week 2-3 time estimates still hold with the mood display, song links, `unclear` handling and an extra bucket to curate? | 2026-11-01, PRD drafting | Week 2: check once the TDD is finalized | Open. No hours changed yet. |

## Resolved

| # | Question | Raised | Resolved | Outcome |
|---|----------|--------|----------|---------|
| Q3 | Build a stable frontend with dummy data first, or backend first? | 2026-10-03, Week 1 planning | 2026-10-03 | Backend-first with a vertical slice each week. A throwaway static page in Week 1; walking skeleton with `FakeClassifier` in Week 2; real LLM in Week 3. The UI is trivial in V1, the risk is in the backend, and the project's goal is backend learning. |
| Q4 | Is scope and docs a "Week 0"? | 2026-10-03, Week 1 planning | 2026-10-03 | No. Week 1 is scope and docs, and the plan stays at six weeks. Keep the PRD to one page and treat the TDD as a first draft to revise in Week 2. |

## Rules

- Each question gets an ID, the stage it was raised, and the stage it blocks.
- A question moves to Resolved only when you've answered it in your own words. The answer is recorded under Outcome and also goes into the TDD "Key decisions" section.
- Any decision that comes out of a question also goes into the Decision Log.
- A week isn't closed until every question that blocks it is resolved.
- Ideas that belong to a later version are parked here too, tagged with their target version, for example `[V3]`.
