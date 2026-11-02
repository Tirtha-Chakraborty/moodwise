# V1 TDD

**Status:** Draft in progress (2026-11-02): mood taxonomy decided; the other entries are still TBD. First draft in Week 1, finalized in Week 2.

## Constraints

- Go
- Fixed mood buckets, classified by an LLM API
- Hand-curated flat JSON dataset
- No embeddings and no database

## Key decisions and why

Each entry gives the decision, the rejected alternative, and what breaks if it's wrong. Every decision here also gets a row in [decision_logs.md](../decision_logs.md).

### Mood taxonomy (Q1, Q8)

- **Decision:** ten fixed moods, defined once in a single embedded JSON file (path settled with the Week 2 layout). Each mood has a `key` (lowercase ASCII slug, unique, stable, and never `unclear`), a `label` for display, and a one-line `definition`. Code contains no mood literals; a mood name appears only in the taxonomy file, the dataset tags, and the tests and eval. The loader checks every dataset tag against the taxonomy at startup, so a bad tag stops the app from booting. The classifier prompt and the enum validation are built from the taxonomy keys and definitions, and templates show the `label`. `unclear` is not a mood: it is a code constant for the classifier's no-mood output.

  | key | definition |
  | --- | --- |
  | `happy` | Content, cheerful or joyful about how things are right now. |
  | `sad` | Grief, sorrow or a low mood, with no blame or grudge. |
  | `angry` | Frustration, irritation or rage, often aimed at someone or something. |
  | `calm` | Quiet and tranquil; wanting to pause and absorb the moment. |
  | `anxious` | Tense and stressed about an uncertain future, often expecting the worst. |
  | `nostalgic` | A bittersweet longing for a past moment or time that still matters, whether the memory is happy or painful. |
  | `lost` | Unsure what you feel, numb or blank, disconnected from yourself. |
  | `love` | Romantic affection and longing: falling for someone, or basking in those feelings. A positive sentiment. |
  | `heartbreak` | The pain of love gone wrong: rejection, betrayal, unrequited love or a breakup. Hurt and loss, not rage. |
  | `energetic` | Fired up and ready to act; drive or anticipation, like a fresh start, a new project or a gym goal. |

  Love is romantic only in V1; affection for family or friends has no bucket. `heartbreak` is separate from `love` so that a heartbroken visitor gets results about the pain, not a mix with happy love songs.
- **Rejected alternative:** hardcoding mood names in each place that uses them; numeric ids instead of readable keys; one `love` bucket covering heartbreak; an LLM-returned tone score within a mood (a second dimension that is unreliable to produce, can't be checked by the eval, and doubles curation).
- **What breaks if wrong:** a mood name appears in about seven places (dataset tags, prompt, enum validation, catalog lookup, templates, eval and tests, docs). Renaming a key after curation means editing every tag for it, and without the startup check a typo'd tag silently drops an item from results with no error. If tests or the eval read their expected moods from the taxonomy file, a wrongly renamed or deleted key changes both sides and the tests still pass, so they hardcode the keys on purpose. Vague definitions blur the buckets, since the prompt is built from them.

### `Classifier` is an interface (Q2)

- **Decision:** TBD
- **Rejected alternative:** TBD
- **What breaks if wrong:** TBD

### Spend guard mechanism (Q5)

- **Decision:** TBD
- **Rejected alternative:** TBD
- **What breaks if wrong:** TBD

### Song link format (Q9)

- **Decision:** each song item carries `title`, `artist` and `urls`, a map keyed by service. The only allowed keys are `spotify` and `youtube`. Values are direct track URLs, not search URLs. At least one URL is required, not both. The loader validates at startup that the keys are in the allowed set, every URL is `https`, and the host matches its service (`open.spotify.com` for `spotify`; `youtube.com` or `youtu.be` for `youtube`). Links are link-out only: no embedded players and no music service API in V1. Every link is checked by hand before launch, alongside the mood-tag review.

  ```json
  { "type": "song", "mood": "nostalgic", "title": "...", "artist": "...",
    "urls": { "spotify": "https://open.spotify.com/track/...", "youtube": "https://www.youtube.com/watch?v=..." } }
  ```
- **Rejected alternative:** search URLs (never break, but land on a results page and may pick the wrong version); a single `url` field (no fallback, and no label for the service); a list of URLs (the template would have to guess the service from the URL); requiring both services (blocks curation when a track is missing on one, and doubles the lookup).
- **What breaks if wrong:** direct links rot, since tracks get removed or region-locked, so a dead link is a broken result with no error. The second service is the fallback, and the pre-launch hand check catches what is already dead. A typo in a service key would show an unlabeled or missing link, so the key set is validated like the mood taxonomy. A URL pasted into the wrong field would send the visitor to the wrong place, which the host check prevents. Storing `title` and `artist` leaves room for a music API to replace hand-copied links later (Q13).

## Design

TBD

## Testing

TBD

## Observability

TBD
