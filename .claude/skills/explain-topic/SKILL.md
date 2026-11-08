---
name: explain-topic
description: Explain any topic in simple words, using one easy example that runs through the whole explanation so each point connects to the overall idea or flow. Works for a TDD section, a decision, a piece of code, a concept or background context. Use when the user asks to explain or understand something, wants to get familiar with a context, or says they can't connect the points.
disable-model-invocation: true
---

# Explain a topic

Explain what the user names so that every point is connected to the whole. The topic can be anything: a TDD section or decision, a doc, existing code, a concept, or background the user wants to get familiar with. It is not limited to drafting. The user is learning backend engineering, so the goal is that they can see where each point sits and why it exists, not just what it says.

The argument is the topic. If none is given, use the topic under discussion, and say which one you picked.

## 1. Ground yourself first

Read what defines the topic before explaining it: the relevant docs (TDD, PRD, decision log, `docs/repo_structure.md`), or the code itself if the topic is code. Explain what the sources say, and mark anything that is only your recommendation as a recommendation. If the sources contradict each other, name the conflict instead of smoothing it over. If the topic is a general concept with no project source, say so and explain it in the context of how Moodwise would use it.

## 2. Pick one simple example and keep it for the whole answer

Choose a small, everyday example that makes the idea easy to picture: a shop with a shelf and a shopkeeper, a library, a post office. Then:

- Introduce it in one or two sentences and say which real parts it stands for (for example, "the shelf is the dataset file, the shopkeeper is the `Catalog`").
- Use it again when explaining each point, so the points read as parts of one story and not a list of separate facts.
- Where the example stops matching reality, say so in one line, so the user does not learn a wrong idea from it.
- If the topic is in Moodwise's own flow, also give a concrete project example, such as one real phrase going through the system.

## 3. Start with the whole picture

Open with the story of the topic from start to end, in a short paragraph or at most five numbered steps:

- For a Moodwise topic, use the project's real flow (startup, then a visitor request, then the page) and say which step the topic belongs to. If it spans startup and request time, say so.
- For code, follow what happens from the entry point to the result.
- For a concept, follow its life: why it is needed, what it does, what breaks without it.

Tell the story in the example's terms first, then map it to the real names.

## 4. Walk through the points, each with its context

Use a table or a short list, one row per point, with three parts:

1. **The point**, in the source's own name for it, so the user can find it in the docs or code.
2. **Where it sits in the flow:** the step it affects, and what comes before and after it.
3. **In simple words:** what it is, why it exists, and what goes wrong without it, in one to three sentences, tied back to the example.

Rules for the wording:

- Plain words. When a technical term is needed, use it once with a short meaning next to it, then keep using it, so the user learns the term.
- Never list a point without its place in the flow. If a point has no clear place, say that, because it may be a sign the design is vague there.
- Keep real component names as they appear in the docs or code (`Matcher`, `Catalog`, the loader).
- Do not add new decisions, new scope or later-version ideas. If one comes up, say it belongs to a later version and where it would be parked.

## 5. Flag what the explanation turned up

If explaining the topic shows a stale statement, a conflict between docs, or an unlogged decision, say so briefly at the end and name the file that should change. Do not edit anything unless the user asks.

## 6. Check understanding

End with one question that checks the "why", not the "what", following the working style in `CLAUDE.md`. Do not move on to the next piece until the user has answered it.

## Output

Text in the chat only. Write nothing to disk.
