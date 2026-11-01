# Moodwise: Project Plan

## Overview

Moodwise is a mood-to-media matcher. A user types a phrase (a lyric, a raw feeling, a single word like "nostalgic") and Moodwise reads the emotional tone behind it, then returns a small, curated set of songs, quotes, books, and movies that resonate with that exact feeling.

The bet: emotion is a better filter for discovery than category or popularity. Sometimes you don't want to search for what you want. You want to describe how you feel and let something find you.

Under the hood, the project is a study in matching unstructured human language to structured media. It starts with simple mood classification and evolves into real semantic (embeddings-based) similarity search as the project matures. It is being built as a personal backend-learning project, at roughly 3-4 hours a week, using Claude and Codex as development accelerants, with each version deliberately scoped to teach a specific backend skill rather than just ship features fast.

## V1: Minimum Lovable Product (current)

Goal: prove the core loop works, end to end, with minimal moving parts, built in Go.

- User types a phrase into a simple form.
- Phrase is sent to an LLM via API, which classifies it into one of 6-8 fixed mood buckets (happy, sad, nostalgic, calm, angry, hopeful, etc.), or "unclear" if it describes no feeling.
- A small, hand-curated dataset (50-100 items: songs + quotes only, stored as flat JSON) is filtered by that mood bucket.
- Results are displayed on a single results page: the detected mood, songs and quotes in separate sections, with songs linking out.
- Deployed live, even if ugly. Includes a feedback form link and basic cost restrictions.

**What it teaches:** basic API design, calling an external LLM API, simple filtering logic, shipping something real.

**What it deliberately skips:** real matching intelligence, multiple categories, persistence, scale, full rate limiting (V4). The V1 spend guard stays in, and only full rate limiting is out.

## V2: Real Matching

Goal: replace fixed mood buckets with actual semantic understanding, and bring in a live external data source.

- Swap mood buckets for embeddings: vectorize the user's phrase and the dataset items, match by cosine similarity instead of discrete categories.
- Add a second content category pulled from a live external API (e.g. movies via TMDB, books via Open Library) instead of static data.
- Add a caching layer so repeated phrases and API calls aren't recomputed every time.
- Handle the external API's failure modes gracefully (timeouts, empty results, rate limits).

**What it teaches:** embeddings / semantic search, integrating a third-party live API, caching strategy, graceful degradation.

**What it deliberately skips:** concurrency, robust resilience patterns, a real persistence layer.

## V3: Production-Grade Backend

Goal: the version that actually mirrors real backend engineering work.

- Add remaining categories (pictures, more quote/book sources), each standardized as a pluggable "source handler."
- Make external calls async/parallel instead of sequential: stream results back as each source responds, rather than making the user wait for the slowest one.
- Add retries with backoff and a basic circuit breaker for flaky external sources.
- Move from flat files to a real database (Postgres) for storing submissions and history.
- Add structured logging and basic metrics (latency per source, cache hit rate).

**What it teaches:** concurrency, resilience patterns, observability, real data modeling. The strongest interview material in the whole project.

**What it deliberately skips:** user accounts, personalization, public scaling concerns.

## V4: Polish & Personalization

Goal: turn a working backend into a product people actually want to return to.

- Lightweight auth, so users can save past results or build a "mood history."
- Personalization: recommendations improve over time based on a user's past inputs (even something simple like weighting previously liked categories).
- Rate limiting on your own API to handle real public traffic responsibly.
- Performance pass under concurrent load; polish the frontend into something genuinely shareable.
- Optional: let users submit their own phrase -> result pairs to grow the dataset organically.

**What it teaches:** auth, lightweight personalization/recommendation logic, handling real public traffic, product thinking beyond pure backend.

Each version above is a complete, shippable product on its own. The project is never left half-broken between stages.
