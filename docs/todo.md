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
