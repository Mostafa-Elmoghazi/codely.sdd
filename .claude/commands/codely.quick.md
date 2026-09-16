---
description: Interview, plan, implement, and open a draft PR for a small well-understood change — no separate spec or plan review
argument-hint: [ticket-id] [brief description]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.quick

> **Scope guardrail:** for small, well-understood changes only — folds spec and plan into one `quick-plan.md` and skips their separate review cycles. Within `.specs/<ticketId>_<slug>/`, writes only `quick-plan.md` and, if deferred items surface, `follow-up.md`. Plan approval gates the Phase 9 branch creation and quick-plan commit; implementation approval gates the Phase 13 commit, push, and draft PR. Outputs are the quick plan, the application code for this change, updates to any already-existing app/package `CLAUDE.md` that change makes stale (never creates a new one — that's `/codely.sys-init`'s job), and — after both approvals — a commit history, push, and draft PR on one new branch.

Turn a ticket ID + brief description directly into a reviewed, working implementation — interview, write a single-phase `quick-plan.md`, implement it, and open a draft PR, all in one run. Follow the phases in order. No git changes before the relevant approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

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
- Task tracker config present: !`test -f .claude/includes/task-tracker.md && echo yes || echo no`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve inputs

**Input text** for this command = the brief description.

@.claude/includes/resolve-ticket-inputs.md

@.claude/includes/worktree-ticket.md

@.claude/includes/ticket-source.md

## Phase 1 — Load the ticket ID format

1. Read `.claude/includes/ticket-format.md`.
2. **Missing** (first run): tell the user, then AskUserQuestion — "describe the format, I'll draft the file" vs. "I'll set it up myself, stop here." If they describe it, ask the scheme (JIRA/GitHub/Linear/none/other), write the file, continue. Otherwise stop.
3. **Present:** extract the pattern/regex and examples.
4. Validate the candidate ticket ID against it.
5. **No match** (or no candidate): show the expected format, AskUserQuestion — corrected ID / use anyway / no-ticket fallback. Never guess.

## Phase 2 — Check for an existing ticket

1. Glob `.specs/` for `<ticketId>_*`. None → go to Phase 3.
2. Found, with `spec.md` or `plan.md` but no `quick-plan.md` (full flow already in progress for this ticket) → tell the user, AskUserQuestion — switch to the full flow instead (stop here, point to `/codely.plan` or `/codely.implement` as appropriate) / proceed with quick anyway (confirm — unusual) / cancel.
3. Found, with `quick-plan.md` → read + summarize it, AskUserQuestion — Resume/revise (reuse slug, skip interview) / Start over (reuse slug, redo interview) / Cancel. Either way skip Phase 3 and reuse the existing slug.

## Phase 3 — Generate the slug (new tickets only)

Inside a ticket worktree, skip this phase — reuse the worktree's slug.

@.claude/includes/slug.md

## Phase 4 — Codebase exploration & size check

@.claude/includes/claude-md-read-first.md

1. Glob/Grep/Read the feature area — similar features, conventions, tests, related specs. Don't ask the user what the code already answers.
2. Rough-estimate files touched and complexity. Clearly past a quick fix (well beyond ~10 files, cross-cutting/architectural, many open unknowns) → tell the user, AskUserQuestion — switch to `/codely.spec` instead (stop here) / continue with quick anyway.

## Phase 5 — Interview the user (rounds)

**Mandatory — don't proceed until scope, edge cases, requirements, technical approach, and constraints are all clear.**

Start from the open questions raised by Phase 0 + Phase 4 findings — this interview covers what a spec interview and a plan interview would each cover, combined. Then:

@.claude/includes/interview.md

Points that matter but shouldn't block this change → tell the user they'll go to a follow-up file, and hold them for Phase 6.

## Phase 6 — Deferred items

1. Phase 5 surfaced deferred points → tell the user they'll be captured for later triage rather than folded into this change.
2. Create (or append to) `.specs/<ticketId>_<slug>/follow-up.md` from `.claude/templates/follow-up.md` with those items.
3. These get triaged one at a time later via `/codely.follow-up`.

## Phase 7 — Write the quick plan

1. Fill `.claude/templates/quick-plan.md` placeholders.
2. Mandatory: Overview (what's being requested, why, what success looks like, and background that led to this request), Approach (high-level strategy and key technical decisions — how this will be implemented), Tasks, Key Files, Acceptance Criteria (checklist that defines "done"), QA Plan (checklist of test cases, ticked off during implementation and by `/codely.qa` if run afterward).
3. Optional (omit a section entirely when empty): Notes (what this explicitly doesn't cover, tagged `[Out of scope]`; important points confirmed by the user during the interview, tagged `[Decision]`).
4. Branch: `<ticketId>/<slug>/quick`, exact order.
5. Write to `.specs/<ticketId>_<slug>/quick-plan.md`, creating the directory if needed.

## Phase 8 — Plan approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/quick-plan.md` (and `follow-up.md` if created) — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 9 — never add an approver name to the file.

## Phase 9 — Branch creation & plan commit

@.claude/includes/git-preflight.md

5. Create/checkout `<ticketId>/<slug>/quick`. If it already exists, check it out and note how far ahead of trunk it is.
6. Stage and commit `quick-plan.md` (and `follow-up.md` if created) plus `.sdlc/backlog.md` if this run changed its status: `Add quick plan for <ticketId>`.
7. Don't push yet — implementation is still pending.

## Phase 10 — Implement the tasks

1. Read the Tasks and Key Files from `quick-plan.md`; Glob/Grep anything not already covered by Phase 4's exploration.
2. Work through unchecked tasks in order.
3. A task's scope or approach is ambiguous and the interview didn't resolve it → don't guess on material decisions: AskUserQuestion, then proceed. Trivial implementation details don't need a round-trip.
4. Follow the conventions already in the files/modules you touch.
5. Mark a task `- [x]` in `quick-plan.md` as soon as its done-criteria is met.
6. Necessary work surfaces outside the task list → don't do it silently. Note it in `follow-up.md` (create from `.claude/templates/follow-up.md` if needed, or append) and flag it to the user.
7. @.claude/includes/claude-md-sync.md

## Phase 11 — Verify

1. Detect the project's build/lint/test commands: `CLAUDE.md` first, then `package.json` scripts / `Makefile` / equivalent. Still unclear → ask the user once rather than guessing at commands that might not exist.
2. Run them. Failures → fix and re-run.
3. A failure exposes a gap in the plan itself, not just a bug in the new code → surface it to the user rather than silently deviating from the approved quick plan.
4. UI-facing changes → exercise the feature in a running app if you're able to; otherwise tell the user this still needs manual verification.

## Phase 12 — Implementation approval loop

1. Summarize the implementation — files touched, diff stat, verification results.
2. Approve or request changes, loop until approved.
3. Approval is a chat-only gate for Phase 13 — never add an approver name to any file.

## Phase 13 — Commit, push, draft PR

**Don't start until Phase 12 approval is explicit.**

1. Stage the changed application files plus `quick-plan.md` (checkbox updates).
2. Commit: `Implement <ticketId>: <title>`.
3. Push the branch, setting upstream if it has none.
4. `follow-up.md` exists with any Open or pending items → the PR body must also include a `- [ ] Follow-Up Triaged` checkbox (unchecked, ticked by `/codely.follow-up` once every item is resolved). No `follow-up.md`, or no items yet → omit it; `/codely.follow-up` adds it later if needed.
5. Trunk = `CLAUDE.md`'s choice, else `main`, else `master`. `gh` available → `gh pr create --draft` (base = trunk, head = `<ticketId>/<slug>/quick`, title referencing the ticket, body linking to `quick-plan.md`, noting this is a combined plan + implementation PR, and including a `- [ ] QA Done` checkbox plus the `- [ ] Follow-Up Triaged` checkbox per step 4 if applicable). `gh` unavailable → tell the user to push and open the PR manually, including the same checkboxes.
6. Confirm: quick-plan path, files changed, commit(s), PR URL (or manual-PR instructions), and which gate checkboxes this PR carries.

## Phase 14 — Next steps

1. Tell the user to get the PR reviewed and merged.
2. Mention `/codely.qa` as an optional final verification pass before or after merge, then `/clear`.
