---
description: Turn an approved spec into a phased implementation plan and open it for review
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(git:*), Bash(gh:*)
disable-model-invocation: true
---

# codely.plan

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.specs/<ticketId>_<slug>/`. Outputs are the plan file (and a follow-up file, if deferred items surface) — and, after approval, a commit + push + draft PR on the ticket's existing docs branch. This command produces a technical plan only; it makes no code changes.

Turn an approved spec into a phased implementation plan at `.specs/<ticketId>_<slug>/plan.md`, then commit, push, and open it as a draft PR for review. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve the ticket

1. Read `CLAUDE.md` if present.
2. Current branch matches `<ticketId>/<slug>/feature` (validate `ticketId` against `.claude/includes/ticket-format.md` if present): extract `ticketId` and `slug` — this is the docs branch.
3. Current branch matches `<ticketId>/<phaseSlug>` (two segments, no `/feature` suffix): extract `ticketId`, but note this is a phase branch, not the docs branch — it must switch back (step 5).
4. On trunk, or neither pattern matches: use `$ARGUMENTS` as the ticket ID if given, else — inside a ticket worktree (see `.claude/includes/worktree-ticket.md`) — the worktree's ticket ID, otherwise ask the user. Don't guess.
5. Locate `.specs/<ticketId>_*`. Missing → tell the user no approved spec exists for this ticket, stop, recommend `/codely.spec`. Found → the directory's slug is the source of truth from here on. If not already on `<ticketId>/<slug>/feature`: dirty working tree → stop, AskUserQuestion (commit / stash / discard / abort); otherwise switch to it — it should already exist from `/codely.spec`, so warn the user before creating it if it's unexpectedly missing.

## Phase 1 — Load context

1. Read `.specs/<ticketId>_<slug>/spec.md`.
2. `.specs/<ticketId>_<slug>/system-design.md` present → read it too.
3. `.specs/<ticketId>_<slug>/design.md` present → read it too, along with its referenced `design/*.html` mockups — these define the screens, components, and states to plan for.
4. `.specs/<ticketId>_<slug>/plan.md` already exists → summarize it, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate) / Cancel.

## Phase 2 — Size the work

@.claude/includes/claude-md-read-first.md

1. Glob/Grep/Read the implicated codebase areas to rough-estimate files touched.
2. Under ~10 files and low complexity: recommend `/codely.quick` instead — AskUserQuestion: switch to `/codely.quick` (stop here) / continue with a plan anyway.
3. Still small but past that bar: AskUserQuestion — single-phase plan or multi-phase plan.
4. Clearly larger: proceed straight to a multi-phase breakdown.

## Phase 3 — Codebase exploration

Glob/Grep/Read the feature area(s) — existing patterns, conventions, related modules, tests. Don't ask the user what the code already answers.

## Phase 4 — Interview the user (rounds)

**Mandatory — don't draft phases until the technical approach, phase breakdown, and any out-of-scope calls are confirmed.**

Start from the open questions raised by spec gaps + exploration findings + the Phase 2 sizing outcome. Then:

@.claude/includes/interview.md

Record any "you decide" call under Notes, tagged `[Decision]`. Points that matter but shouldn't block these phases → tell the user they'll go to a follow-up file, and hold them for Phase 6.

## Phase 5 — Draft the phase breakdown

1. Define each phase: title, branch slug (slug rule per `.claude/includes/slug.md`, applied to the phase title), ordered `- [ ]` tasks each with done-criteria, key files.
2. A single-phase plan still uses the `Phase 1: <title>` heading.
3. Sequencing is fixed, not per-plan content: each phase's PR bases on `main` if the previous phase is already merged, otherwise stacks on the previous phase's branch.
4. A phase's Key Files fall under an app/package that has its own `CLAUDE.md`, and this phase's work looks likely to make it stale (new files it should list, a changed convention) → note that file alongside Key Files. This is a flag for `/codely.implement` to act on, not a change made here — this command never touches application code.

## Phase 6 — Deferred items

1. Phase 4 surfaced deferred points → tell the user they'll be captured for later triage rather than added to these phases.
2. Create (or append to) `.specs/<ticketId>_<slug>/follow-up.md` from `.claude/templates/follow-up.md` with those items.
3. These get triaged one at a time later via `/codely.follow-up`.

## Phase 7 — Write the plan

1. Fill `.claude/templates/plan.md` placeholders.
2. Mandatory: Approach (high-level strategy and key technical decisions — how the spec's requirements will be implemented), Phases (each with Branch/Tasks/Key Files — repeat the `### Phase N: {{title}}` block for each additional phase beyond the first; phases run in order, each phase's PR bases on `main` if the previous phase is already merged, otherwise stacks on the previous phase's branch), QA Plan (checklist of test cases, ticked off by `/codely.qa`).
3. Optional (omit a section entirely when empty): Notes (items explicitly confirmed out of scope during planning, tagged `[Out of scope]`; important points confirmed by the user during planning, tagged `[Decision]`).
4. Write to `.specs/<ticketId>_<slug>/plan.md`.
5. Remind the user: no application code has changed — only the plan (and possibly follow-up) file.

## Phase 8 — Approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/plan.md` (and `follow-up.md` if created) — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 9 — never add an approver name to the file.

## Phase 9 — Commit, push, draft PR

**Don't start until Phase 8 approval is explicit.**

1. Stage only the plan file(s) written above (`plan.md`, `follow-up.md` if created).
2. Commit: `Add implementation plan for <ticketId>`.
3. Push the current (docs) branch, setting upstream if it has none.
4. Trunk = `CLAUDE.md`'s choice, else `main`, else `master`. `gh` available → `gh pr create --draft` (base = trunk, head = the docs branch, title referencing the ticket, body noting this reviews the spec + plan docs only, no code changes). `gh` unavailable → tell the user to push and open the PR manually.
5. Confirm: plan path, PR URL (or manual-PR instructions). Tell the user: get this PR approved, then run `/clear` then `/codely.implement`.
