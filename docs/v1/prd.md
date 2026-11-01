# Moodwise V1: Product Requirements

Status: Draft 2 · 2026-11-01

## Problem

People often can't name what they want to listen to or read, but they can describe how they feel. Moodwise tests whether emotion is a better discovery filter than category or popularity: type a phrase, get songs and quotes that match the feeling behind it.

## Audience

Anyone who opens the public URL. V1 doubles as a portfolio piece, so a visitor needs no context and there are no accounts.

## User flow

1. The visitor types a phrase (a feeling, a lyric, a single word like "nostalgic") into one form.
2. The system works out the mood behind it and shows it ("Sounds nostalgic").
3. One results page shows up to 3 songs and up to 2 quotes in separate sections. Each song links out to an external service.
4. If the phrase doesn't describe a feeling (gibberish, off-topic, instructions aimed at the system), the visitor sees a friendly "try describing how you feel" message and no results.
5. If something fails, the visitor sees a clear message, never a crash or a blank page.
6. A feedback form link appears on the home page and the results page.

## What the visitor gets

A small, hand-picked set of songs and quotes (50-100 items), grouped under 6-8 fixed moods, including one for uncertain or unsettled feelings. Similar feelings may return the same picks, and repeat visits draw different items at random from the same small pool.

## Scope

**In:** single-phrase form; mood classification into 6-8 fixed moods; hand-curated songs and quotes; results page with detected mood, two sections and song links; public URL; feedback form link; bounded cost (see Cost and abuse).

**Out (deliberately):** books and pictures (books V2, pictures V3); embeddings and semantic matching; external data sources; caching; a database; accounts and personalization; full rate limiting (V4).

## Success criteria

- **End to end:** a typical phrase returns results in under 3 seconds at the public URL.
- **Sensible classification:** on a hand-labeled set of ~20 typical phrases, at least 90% land in a mood a reasonable person would accept (accepted moods written before running). This is a sanity check for gross errors (a heartbreak phrase must not return "happy"), not a test of fine distinctions.
- **Honest dataset:** every item's mood tag is reviewed by hand before launch.
- **Non-feelings are blocked:** gibberish, off-topic input and instruction-style input return no results and the friendly message. All test phrases of this kind (5-8) must pass.
- **Fails gracefully:** empty or over-long input and LLM failures each show a sensible message.
- **Bounded cost:** see below.
- **Feedback:** a handful of people outside my circle have tried it and used the form.
- **Learning:** I can explain every design decision in the TDD without notes.

## Cost and abuse

The public endpoint spends money on every request, so V1 must have restrictions that bound total spend and limit abuse from a single visitor. The specific mechanisms are decided in the TDD before the build starts (Open Question Q5).

## Known limitations

- One mood per phrase: mixed feelings are flattened, and similar feelings (e.g. "heartbroken" and "lonely") can return the same results.
- The dataset is small, so results repeat over time.
- A mood with too few items shows fewer results; it is never padded with items from another mood.
- Matching is a lookup in my own tagged list, not a search of real-world catalogs.
- Classification is by an LLM and can be wrong; its output is validated, but it is not guaranteed correct and the system is not claimed to be injection-proof.
- Phrases are not stored by the app, but they are sent to a third-party AI provider; the page says so.
- Song links may break over time.
