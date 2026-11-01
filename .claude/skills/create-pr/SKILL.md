---
name: create-pr
description: Create a pull request for the current branch, after checking that the docs hygiene rules in CLAUDE.md are met. Use when the user asks to open, create or raise a PR.
disable-model-invocation: true
---

# Create a PR

Open a pull request for the current branch. Work through the steps in order, and stop to tell the user if a step fails.

## 1. Check the branch

- Run `git status` and `git branch --show-current`.
- Never open a PR from `main`. If on `main`, create a branch first, named for the change (for example `docs/project-scaffolding` or `feat/classifier`).
- If the working tree has uncommitted changes, show them and ask whether to include them in a commit. Don't commit unrelated files.

## 2. Check the docs hygiene

Read the diff against `main` (`git diff main...HEAD`) and check each item in CLAUDE.md's "Project hygiene" section:

- A decision made in this PR has a row in docs/decision_logs.md.
- An open question parked in this PR is in docs/open_questions.md, and a resolved one is removed and logged as "(Resolves Q#.)".
- A follow-up is in docs/todo.md, and the code has no TODO or FIXME comments.
- A plan change shows before and after in the PR description.
- The docs are consistent with each other and with the code, with nothing stale.

Fix gaps in the branch before continuing. Not every PR has a decision, question or to-do, so "none" is a valid answer.

## 3. Check the code

If the PR contains Go code, run `go vet ./...` and `go test ./...` and report the result. Don't open the PR if either fails, unless the user says to. If there is no code yet, skip this step and say so.

## 4. Write the title and description

- Title: short, imperative, under about 70 characters, for example "Add classifier interface and fake".
- Description: follow `.github/pull_request_template.md`, with a Summary that says what changed and why, and the Docs checklist filled in. Write "none" for items the PR has nothing of.
- Write for a reviewer who has not seen the conversation.
- End with the attribution line for pull requests that the session's system reminder gives, if there is one.

## 5. Push and open the PR

- Push the branch with `git push -u origin <branch>`.
- If `gh` is installed, open the PR against `main` with `gh pr create`, and give the user the link.
- If `gh` is not installed, give the user the `https://github.com/<owner>/<repo>/pull/new/<branch>` link and the title and description ready to paste. Say clearly that the PR has not been opened yet.
