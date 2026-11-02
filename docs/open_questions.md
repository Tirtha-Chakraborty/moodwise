# Moodwise V1: Open Questions Log

Last updated: 2026-11-02

Only unresolved questions live here. When a question is resolved, it is removed from this file and logged in [decision_logs.md](decision_logs.md) as a row ending with "(Resolves Q#.)".

## Open

| # | Question | Raised | Stage it blocks | Status |
|---|----------|--------|-----------------|--------|
| Q2 | Why is `Classifier` an interface, and what would change in the handler if it called the LLM SDK directly? | 2026-10-03, Week 1 planning | Week 2: must be answered before the walking skeleton | Open, to be answered in the TDD "Key decisions" section |
| Q5 | Which mechanism(s) implement the V1 spend guard (provider spend cap, input length cap, global daily counter, per-IP limit)? The PRD states only the requirement. | 2026-11-01, PRD drafting | Week 2: close in the TDD before the build starts (built in Week 4) | Open. Candidate layers discussed, nothing chosen. On resolution, update the PRD's cost section and add a Decisions Log row. |
| Q10 | Do the Week 2-3 time estimates still hold with the mood display, song links, `unclear` handling and an extra bucket to curate? | 2026-11-01, PRD drafting | Week 2: check once the TDD is finalized | Open. No hours changed yet. The mood cap rose to 10 on 2026-11-02, so include that. |
| Q11 | `[V2]` Which finer-grained moods belong in the V2 taxonomy? Parked ideas: hopeful, melancholy, demotivated, aloof, spiritual (devotion to a higher presence; closer to a theme than a feeling, and curation is faith-sensitive). | 2026-11-02, Q8 discussion | V2: taxonomy review | Open. Not for V1; 10 is the final V1 count. |
| Q12 | `[V2]` Should matching carry a tone or intensity dimension within a mood (for example love: falling in love versus the sting of it; nostalgia: happy versus painful memories; lost: disconnected versus depleted)? V1 uses separate buckets instead of an LLM-returned score. | 2026-11-02, Q8 discussion | V2: matching design (embeddings give graded closeness) | Open. Not for V1. |
| Q13 | `[V2]` Should song links come from a legitimate music API (search or lookup by title and artist) instead of hand-copied URLs? Fits V2's live-API integration and caching work; V3 source handlers are the fallback. V1 prepares by storing `title` and `artist` on every song item. | 2026-11-02, Q9 discussion | V2: scope check when the live-API work is planned | Open. Not for V1; the V1 PRD lists external data sources as out of scope. |

## Rules

- Each question gets an ID, the stage it was raised, and the stage it blocks.
- IDs are never reused or renumbered. Q1, Q3, Q4, Q7, Q8 and Q9 are resolved and live in the Decision Log; there is no Q6.
- A question is resolved only when you've answered it in your own words. The answer goes into the TDD "Key decisions" section, and a row ending with "(Resolves Q#.)" goes into the Decision Log. Then remove the question from this file.
- A week isn't closed until every question that blocks it is resolved.
- Ideas that belong to a later version are parked here too, tagged with their target version, for example `[V3]`.
