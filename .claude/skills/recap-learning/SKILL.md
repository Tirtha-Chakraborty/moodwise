---
name: recap-learning
description: Write a copy-pastable learning recap of this session (questions asked, the user's answers, corrections, and the takeaway) for the user's private learning notes. Use when the user asks to recap, summarise their learning, or write learning notes.
disable-model-invocation: true
---

# Recap learning

Produce text the user pastes into their private `learnings/` folder. The folder is excluded from git, so this skill only outputs text. Never write to `learnings/`, never stage it, and never mention its contents in any committed file.

The recap is the user's record of what they learned, so it must be faithful to what they said. It is not a summary of what the assistant did.

## 1. Gather the material

Go through the conversation and list each teaching moment: a question the assistant asked to check understanding, a design choice the user made or argued for, and any point where the assistant corrected them.

- Quote the user's answers as they wrote them, in quotation marks. Fix nothing and tidy nothing, so they can see how they actually put it.
- Record each correction as: what the user said, what was wrong or missing, and the gap in understanding. If an answer was right, say so and note anything that was added.
- Skip chores with no learning in them (renaming a file, formatting).

If the session is long, early exchanges may have been summarised away. Say so in the output rather than reconstructing what was said, and remind the user to run the recap more often in long sessions, ideally after each piece of work.

## 2. Group into entries

One entry per concept, not one per message. Label each as a **project concept** (specific to Moodwise, such as the taxonomy or the song schema) or a **core concept** (general backend or Go knowledge, such as interfaces or `git clean`). Suggest a file name for each, for example `learnings/project-concepts/w1-taxonomy-single-source.md`. If a concept already has an entry from an earlier recap, give only the new material to append and say which file it goes in.

## 3. Use this entry format

1. **Concept:** project or core, and what it is.
2. **The idea in my words:** leave a clearly marked placeholder, `(Rewrite this in your own words)`, with a draft underneath. The user's own wording is the point, so do not present the draft as final.
3. **What I got wrong first:** the corrections from step 1, each with the gap behind it.
4. **Check question and my answer:** the question asked, the user's answer quoted, and what was added.

## 4. Output

- Put each file in its own fenced markdown block, with the suggested path in the line above it, so the user can copy it in one go.
- End with a short session summary: what was decided, what was learned, and what is still open.
- Do not save anything yourself. Tell the user the text is ready to paste.
