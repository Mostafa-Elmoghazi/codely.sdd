---
description: Interview the user into an approved course curriculum, then branch, commit, and push it
argument-hint: [brief course description]
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Bash(git:*)
disable-model-invocation: true
---

# trainer.start

> **Scope guardrail:** never writes lesson content — only drafts `.courses/<slug>/curriculum.md`. Outputs are the curriculum file and — after approval — a new `course/<slug>` branch with it committed and pushed.

Act as a professional trainer. Turn a course brief into an approved curriculum — modules lettered A, B, C…, each with lessons numbered 1, 2, 3… (id `{{module}}.{{lesson}}`, e.g. `A.1`) and a final `(Workshop)` lesson — then cut the course's branch with the curriculum committed and pushed to it. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.courses/` contents: !`ls -1 .courses 2>/dev/null || echo "(.courses/ does not exist yet)"`

## Phase 0 — Resolve the brief

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty → that's the course brief. Empty → ask the user for a one- or two-line pitch of the course. Don't continue without one.

## Phase 1 — Generate the slug and check for an existing course

1. From the brief's likely title: lowercase, `-`/`_` only, no dots, ~5 words/40 chars max.
2. Glob `.courses/` for a matching or near-matching directory.
3. **Found**, with a `curriculum.md` inside: read + summarize it, AskUserQuestion — Resume/revise (reuse slug, skip the interview, jump to Phase 3 to edit) / Start over (reuse slug, redo the interview) / Cancel.
4. **Not found:** go to Phase 2.

## Phase 2 — Interview the user (rounds)

**Mandatory — don't draft the curriculum until level, audience, scope, and depth are all clear.**

Work out what's still open: target skill level (beginner/intermediate/advanced) and audience, learning outcomes, topics in/out of scope, prerequisites, roughly how many modules/lessons and how deep each should go, subject-specific tooling or stack (for technical courses), and whether module workshops should lean toward guided exercises or open-ended projects. Then:

@.claude/includes/interview.md

## Phase 3 — Draft the curriculum

1. Derive the module breakdown (A, B, C…) and, within each, the lesson sequence (1, 2, 3…) from the interview — repeat `- [ ] **Module X — {{title}}**` (B, C, D…) with its own lettered lessons for each additional module. Every module's last lesson is `(Workshop)` — hands-on practice or a small project tying that module's lessons together.
2. Fill `.claude/templates/curriculum.md` placeholders.
3. Mandatory: Overview (what the course covers and what the learner can do by the end), Level & Audience, Curriculum (nothing below the Curriculum heading is edited by `/trainer.start` once approved — only `/trainer.train` writes there after that, ticking each lesson `- [x]` as it's taken). Optional (omit a section entirely when empty): Prerequisites, Decisions.
4. Write to `.courses/<slug>/curriculum.md`, creating the directory if needed.

## Phase 4 — Approval loop

1. Ask the user to review `.courses/<slug>/curriculum.md` — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 5 — never add an approver name to the file.

## Phase 5 — Branch, commit, and push

@.claude/includes/git-preflight.md

5. Create/checkout `course/<slug>`. If it already exists, check it out and note how far ahead of trunk it is (resuming Phase 1's Resume/revise path).
6. Stage and commit `curriculum.md`: `Add curriculum for <slug>` for a new course, `Update curriculum for <slug>` for a revision.
7. Push, setting upstream if none.
8. Confirm: curriculum path, branch name, push result. Tell the user to run `/clear` then `/trainer.train <slug>` to take the first lesson.
