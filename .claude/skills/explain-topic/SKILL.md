---
name: explain-topic
description: Explain a topic, section or set of design points in simple words, placed in the request flow so the user can see where each point fits. Use when the user asks to explain a topic, says they can't connect the points to the overall flow, or asks for a plain-words walkthrough.
disable-model-invocation: true
---

# Explain a topic

Explain what the user names (a TDD section, a decision, a group of points, a concept) so that each point is connected to the whole system. The user is learning backend engineering, so the goal is that they can see where a point sits and why it exists, not just what it says.

The argument is the topic. If none is given, use the topic under discussion, and say which one you picked.

## 1. Ground yourself first

Read the docs that define the topic before explaining it: the TDD section and its decisions, the PRD, and `docs/repo_structure.md` if packages are involved. Explain what the docs say, and mark anything that is only your recommendation as a recommendation. If the docs contradict each other, name the conflict rather than smoothing it over.

## 2. Start with the whole picture

Open with the story of one request or one lifecycle, in a short paragraph or a numbered list of at most five steps:

- Use the project's real flow (startup, then a visitor request, then the page), not a generic one.
- Say which step or steps the topic belongs to. If it spans startup and request time, say so.
- Use an everyday comparison only if it makes the picture shorter, and drop it once the real names are introduced.

## 3. Walk through the points, each with its context

Put the points in a table or a short list, one row per point, with three parts:

1. **The point**, in the docs' own name for it, so the user can find it in the TDD.
2. **Where it sits in the flow:** the step it affects, and what happens before and after it.
3. **In simple words:** what it is, why it exists, and what goes wrong without it. One to three sentences.

Rules for the wording:

- Plain words. When a technical term is needed, use it once with a short meaning next to it, then keep using it, so the user learns the term.
- Never list a point without its place in the flow. If a point has no clear place, say that, because it may be a sign the design is vague there.
- Keep the real names of components (`Matcher`, `Catalog`, the loader) as they appear in the TDD.
- Do not add new decisions, new scope or later-version ideas. If one comes up, say it belongs to a later version and where it would be parked.

## 4. Flag what the explanation turned up

If explaining the topic shows a stale statement, a conflict between docs, or an unlogged decision, say so briefly at the end and name the file that should change. Do not edit anything unless the user asks.

## 5. Check understanding

End with one question that checks the "why", not the "what", following the working style in `CLAUDE.md`. Do not move on to the next piece until the user has answered it.

## Output

Text in the chat only. Write nothing to disk.
