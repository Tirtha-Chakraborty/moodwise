# Moodwise: Nice-to-Have Features

An inventory of small, optional extras that are not part of any version's committed scope. They are worth keeping, but none is vital. Check this list at each milestone wrap-up and pull an item into a version only when there is room and a reason.

Unresolved design questions do not belong here; they go in [open_questions.md](open_questions.md). A feature moves out of this file when it is scheduled into a version's plan (note it in the plan and delete the row).

| # | Feature | Earliest version | Why it might be worth it, and what it depends on | Added |
| --- | --- | --- | --- | --- |
| N1 | Filter results by music service (and later book service), depending on the accounts the visitor has | V4 | Spares visitors links they can't use. Depends on the `urls` map keyed by service in the song schema, which V1 already has. | 2026-11-02 |

## Rules

- Each item gets an ID (N1, N2, ...). IDs are never reused.
- Say what the feature depends on, so it is clear when it becomes cheap.
- Keep entries to a line or two. A feature that needs real design gets an open question or a plan change instead.
