# Moodwise V1: Open Questions Log

Last updated: 2026-11-01

Only unresolved questions live here. When a question is resolved, it is removed from this file and logged in [decision_logs.md](decision_logs.md) as a row ending with "(Resolves Q#.)".

## Open

| # | Question | Raised | Stage it blocks | Status |
|---|----------|--------|-----------------|--------|
| Q1 | What breaks, and in how many places, if a mood bucket is renamed or added after the dataset is half built? (Answer by listing every place a bucket name appears as a string.) | 2026-10-03, Week 1 planning | Week 1: must be answered before curating dataset items and freezing the taxonomy | Open, awaiting your draft |
| Q2 | Why is `Classifier` an interface, and what would change in the handler if it called the LLM SDK directly? | 2026-10-03, Week 1 planning | Week 2: must be answered before the walking skeleton | Open, to be answered in the TDD "Key decisions" section |
| Q5 | Which mechanism(s) implement the V1 spend guard (provider spend cap, input length cap, global daily counter, per-IP limit)? The PRD states only the requirement. | 2026-11-01, PRD drafting | Week 2: close in the TDD before the build starts (built in Week 4) | Open. Candidate layers discussed, nothing chosen. On resolution, update the PRD's cost section and add a Decisions Log row. |
| Q8 | What are the final 6-8 moods and their one-line definitions? Candidate: happy, sad, nostalgic, calm, angry, hopeful, anxious, lost. | 2026-11-01, PRD drafting | Week 1: before dataset curation and the taxonomy freeze (also needs Q1) | Open, candidate proposed |
| Q9 | Which link format for songs: direct track URL (can break) or a search URL (more stable)? Which service? | 2026-11-01, PRD drafting | Week 1: dataset schema | Open |
| Q10 | Do the Week 2-3 time estimates still hold with the mood display, song links, `unclear` handling and an extra bucket to curate? | 2026-11-01, PRD drafting | Week 2: check once the TDD is finalized | Open. No hours changed yet. |

## Rules

- Each question gets an ID, the stage it was raised, and the stage it blocks.
- IDs are never reused or renumbered. Q3, Q4 and Q7 are resolved and live in the Decision Log; there is no Q6.
- A question is resolved only when you've answered it in your own words. The answer goes into the TDD "Key decisions" section, and a row ending with "(Resolves Q#.)" goes into the Decision Log. Then remove the question from this file.
- A week isn't closed until every question that blocks it is resolved.
- Ideas that belong to a later version are parked here too, tagged with their target version, for example `[V3]`.
