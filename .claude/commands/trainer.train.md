---
description: Deliver one lesson of a course (explanation, examples, quiz or project), then record progress and push
argument-hint: [course-slug] [lesson-id, e.g. A.1]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(git:*), Artifact
disable-model-invocation: true
---

# trainer.train

> **Scope guardrail:** only ever touches `.courses/<slug>/curriculum.md` and the target lesson's own folder — never another lesson's files, never application code. Outputs are the lesson's `lesson.html` (and, if chosen, `quiz.md` or `project.md`), the curriculum checkbox/score update, and a commit + push to the course's existing `course/<slug>` branch. No PR — this is content, not a review artifact.

Act as a professional trainer. Deliver the requested (or next unfinished) lesson of an approved course: generate its lesson content, wait for the learner to take it, optionally quiz or project them on it, then record progress and push. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- `.courses/` contents: !`ls -1 .courses 2>/dev/null || echo "(.courses/ does not exist yet)"`

## Phase 0 — Resolve the course

1. Read `CLAUDE.md` if present.
2. Determine the slug: current branch matches `course/<slug>` with a matching `.courses/<slug>/curriculum.md` → use it. Else `$ARGUMENTS`' first token matches an existing `.courses/<slug>/curriculum.md` → use it. Else glob `.courses/*/curriculum.md`: none found → tell the user to run `/trainer.start` first, stop; exactly one → confirm it's the one meant; more than one → list each course's title, AskUserQuestion which one.
3. Not already on `course/<slug>` → `git status --porcelain` dirty → stop, AskUserQuestion (commit / stash / discard / abort); otherwise checkout `course/<slug>`, pulling first if it tracks a remote.

## Phase 1 — Resolve the lesson

1. Read `curriculum.md`; parse every module and lesson with its id, title, and `[ ]`/`[x]` state.
2. `$ARGUMENTS` has a second token matching an existing lesson id (`^[A-Z]+\.[0-9]+$`) → target = that lesson. Given but no match → show the valid ids, ask again.
3. No lesson id given → target = the first unticked lesson in id order (all of Module A before Module B, etc.). Every lesson already `[x]` → congratulate the user, tell them the course is complete, stop.
4. Target lesson already `[x]` → AskUserQuestion: retake it (regenerate content) / take the next unticked lesson instead / cancel.

## Phase 2 — Generate the lesson

1. Derive a lesson slug from its title (slug rule per `.claude/includes/slug.md`).
2. Folder: `.courses/<slug>/<lesson-id>_<lesson-slug>/` (e.g. `.courses/html-basics/A.1_intro-to-html/`) — create it if needed.
3. Build a self-contained `lesson.html` (inline CSS/JS, no external requests): explanation of the concept(s), at least one worked example, code samples where the subject calls for them, a visualization or diagram where it aids understanding (inline SVG, or a `<pre class="mermaid">` block), and 2-4 practice exercises with how to self-check them. Connect it explicitly to the course's stated outcomes and to concepts from prior completed lessons. **`(Workshop)` lesson:** replace the exercises with a hands-on project brief instead — goal, requirements, suggested steps, done-criteria.
4. Publish it via the Artifact tool for a live preview link — session convenience only; the written `lesson.html` is the durable output.

## Phase 3 — Wait for the learner to take the lesson

1. Share the file path and preview link; ask the user to read it (and, for a `(Workshop)` lesson, do the hands-on work) before continuing.
2. AskUserQuestion: I've read/practiced this lesson / not yet. Loop until confirmed — don't assume it happened just because the file exists.
3. Revisions requested → edit `lesson.html`, republish to the same Artifact URL, re-ask.

## Phase 4 — Choose an assessment

Skip this phase for a `(Workshop)` lesson — Phase 3's hands-on work already is its assessment; go straight to Phase 6 with no quiz/project note.

1. AskUserQuestion: Take a quiz / Take a project / Skip assessment for this lesson.
2. **Take a quiz** → Phase 5a. **Take a project** → Phase 5b. **Skip** → Phase 6 with no note.

## Phase 5a — Quiz

1. Write `.../quiz.md`: 5-8 questions covering this lesson's content (mix of multiple-choice and short-answer), each ending in a `**Your answer:**` blank for the learner to fill in directly in the file. No answer key in this file.
2. Tell the user to fill it in and confirm when done. AskUserQuestion: Done / not yet — loop until confirmed.
3. Read the filled-in `quiz.md`, grade each answer against the lesson content, and compute a score (e.g. `7/8`).
4. Report the score, with a brief rationale for any question marked wrong.
5. Carry the score into Phase 6 as `(Quiz: <score>)`.

## Phase 5b — Project

1. Write `.../project.md`: goal, requirements, suggested steps, and done-criteria for a small project scoped to what this lesson alone covers.
2. Tell the user to build it (it doesn't need to live in the course folder) and confirm when done. AskUserQuestion: Done / not yet — loop until confirmed. Ask for a one-line summary of what they built.
3. Carry that into Phase 6 as `(Project: done — <summary>)`.

## Phase 6 — Update the curriculum

1. Edit `curriculum.md`: tick the lesson `- [x]`, appending `(Quiz: <score>)` or `(Project: done — <summary>)` from Phase 5a/5b if one was taken, or leaving the title bare if skipped or a `(Workshop)` lesson.
2. Every lesson under its module is now `[x]` → tick that module's own checkbox too.

## Phase 7 — Commit and push

1. `git status --porcelain`, disregarding `curriculum.md` and this lesson's folder — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Stage `curriculum.md` and the lesson's folder.
3. Commit: `Complete lesson <lesson-id> of <slug>`.
4. Push `course/<slug>`, setting upstream if none.
5. Confirm: files committed, push result.

## Phase 8 — Next steps

1. Lessons remain unticked → name the next unticked lesson's id and title; tell the user to run `/clear` then `/trainer.train <slug> <next-lesson-id>` (or `/trainer.train <slug>` with no lesson id to have it picked automatically).
2. Every lesson now `[x]` → congratulate the user — the course is complete.
