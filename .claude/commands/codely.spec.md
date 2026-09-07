---
description: Draft an approved requirements spec and create its ticket branch
argument-hint: [ticket-id] [brief description]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(git:*)
disable-model-invocation: true
---

# codely.spec

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.claude/includes/ticket-format.md` and `.specs/`. Outputs are the spec file and — after approval — a new local branch with the spec committed to it; `ticket-format.md`, if drafted, is committed immediately as shared project config, independent of the spec's own approval gate. Phase 4 exploration is read-only.

Turn a ticket ID + brief description into an approved spec under `.specs/`, then cut the ticket's local branch with the spec committed to it. Follow the phases in order. No git changes before the spec's approval, except `ticket-format.md` (Phase 1), which is shared project config committed immediately once the user confirms it. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`

## Phase 0 — Resolve inputs

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: first token (minus trailing `:`) = ticket ID, rest = description.
3. `$ARGUMENTS` empty: ask the user for both. Don't continue without them.

## Phase 1 — Load the ticket ID format

1. Read `.claude/includes/ticket-format.md`.
2. **Missing** (first run): tell the user, then AskUserQuestion — "describe the format, I'll draft the file" vs. "I'll set it up myself, stop here." If they describe it, ask the scheme (JIRA/GitHub/Linear/none/other), write the file, then stage and commit it on whatever branch is currently checked out: `Add ticket ID format config`. Otherwise stop.
3. **Present:** extract the pattern/regex and examples.
4. Validate the candidate ticket ID against it.
5. **No match** (or no candidate): show the expected format, AskUserQuestion — corrected ID / use anyway / no-ticket fallback. Never guess.

## Phase 2 — Check for an existing ticket

1. Glob `.specs/` for `<ticketId>_*`.
2. **None:** go to Phase 3.
3. **Found:** read + summarize its `spec.md`, AskUserQuestion — Resume/revise (reuse slug, skip interview) / Start over (reuse slug, redo interview) / Cancel. Either way skip Phase 3 and reuse the existing slug.

## Phase 3 — Generate the slug (new tickets only)

@.claude/includes/slug.md

## Phase 4 — Codebase exploration

Glob/Grep/Read the feature area — similar features, conventions, tests, related specs. Don't ask the user what the code already answers.

## Phase 5 — Interview the user (rounds)

**Mandatory — don't proceed until scope, edge cases, functional/non-functional requirements, and constraints are all clear.**

Start from the open questions raised by Phase 0 + Phase 4 findings, then:

@.claude/includes/interview.md

## Phase 6 — Write the spec

1. Fill `.claude/templates/spec.md` placeholders.
2. Mandatory: Branch, Ticket ID, Summary (what's being requested, why, and what success looks like), Context (background that led to this request), Functional Requirements (testable behaviors the system must implement), Acceptance Criteria (checklist that defines "done").
3. Optional (omit a section entirely when empty): Out of Scope (what this explicitly doesn't cover), User Stories (omit if not user-facing; `"As a ⟨role⟩, I want ⟨capability⟩, so that ⟨benefit⟩."`), Edge Cases, Non-Functional Requirements (performance, security, accessibility, compatibility, etc.), Constraints & Assumptions.
4. Branch: `<ticketId>/<slug>/feature`, exact order.
5. Write to `.specs/<ticketId>_<slug>/spec.md`, creating the directory if needed.

## Phase 7 — Approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/spec.md` — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 8 — never add an approver name to the file.

## Phase 8 — Branch creation and commit

@.claude/includes/git-preflight.md

5. Create/checkout `<ticketId>/<slug>/feature`. If it already exists, check it out and note how far ahead of trunk it is.
6. Stage and commit `spec.md` if it has uncommitted changes: `Add spec for <ticketId>` for a new spec, `Update spec for <ticketId>` for a revision (Phase 2's Resume/revise path). No changes to commit → skip silently.
7. Never push.
8. Confirm: spec path, branch name, current branch. Tell the user to run `/clear` then, optionally, `/codely.sys-design` (architecture/data/API design) and/or `/codely.ui` (screen mockups) — or straight to `/codely.plan` if neither is needed.
