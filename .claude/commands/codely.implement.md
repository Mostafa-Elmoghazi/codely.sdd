---
description: Implement one phase of an approved plan, verify it, and open it as a PR
argument-hint: [ticket-id] [phase-number]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.implement

> **Scope guardrail:** implements only the current phase's tasks from an approved plan — no unrelated refactors, no scope beyond what's listed. Within `.specs/<ticketId>_<slug>/`, touches only `plan.md` (checkbox updates) and, if deferred items surface, `follow-up.md` — never `spec.md`. Outputs are the application code changes for this phase and — after approval — a commit + push + PR on a new phase branch.

Implement the next (or a specified) phase from `.specs/<ticketId>_<slug>/plan.md`, verify it, then commit, push, and open it as a PR for review. One run implements one phase. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve the ticket and target phase

1. Read `CLAUDE.md` if present.
2. Determine `ticketId`:
   - Current branch matches `<ticketId>/<phaseSlug>` (phase branch, two segments, no `/feature` suffix) → extract `ticketId`; remember `phaseSlug` for step 6.
   - Current branch matches `<ticketId>/<slug>/feature` (docs branch), is trunk, or neither pattern matches → use `$ARGUMENTS`' first token if given, else ask. Don't guess.
3. Locate `.specs/<ticketId>_*`. Missing → tell the user, stop, recommend `/codely.spec`. Found → the directory's slug is the source of truth from here on.
4. Read `.specs/<ticketId>_<slug>/plan.md`. Missing → stop, recommend `/codely.plan`.
5. Parse the Phases section: each phase's title, branch slug, task checkboxes.
6. Determine the target phase:
   - `$ARGUMENTS` gives a phase number → use it (validate it exists in the plan).
   - Else, current branch's `phaseSlug` (from step 2) matches a phase's branch → target that phase.
   - Else → the first phase with any unchecked task. All phases fully checked → tell the user, stop, recommend `/codely.qa`.
7. Target phase already has some tasks checked (resuming) → summarize what's done, confirm continuing rather than restarting.

## Phase 1 — Load context

1. Read `.specs/<ticketId>_<slug>/spec.md`.
2. `system-design.md` present → read it.
3. `design.md` present → read it, along with its referenced `design/*.html` mockups — these are the approved markup/structure/states to implement against.
4. `follow-up.md` present → read it. Don't silently redo anything already deferred there.
5. Re-read the target phase's Tasks and Key Files from `plan.md`.

## Phase 2 — Prepare the phase branch

1. `git status --porcelain`. Dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Determine the base per `plan.md`'s sequencing rule: trunk (`CLAUDE.md`'s choice, else `main`, else `master`) if this is the first phase or the previous phase's branch is already merged into trunk; otherwise the previous phase's branch. Check "merged" via `git branch --merged <trunk>` or, if `gh` is available, the previous phase's PR state.
3. Fetch/pull the base if it tracks a remote.
4. Create/checkout `<ticketId>/<phaseSlug>` from that base. Already exists → check it out, note commits ahead of base (resuming).

## Phase 3 — Codebase exploration

Read the phase's Key Files, then Glob/Grep the surrounding area for conventions, patterns, and existing tests not already surfaced by the spec or plan. Read-only.

## Phase 4 — Implement the phase's tasks

1. Work through unchecked tasks in the order listed.
2. A task's scope or approach is ambiguous and the spec/plan doesn't resolve it → don't guess on material decisions: AskUserQuestion, then proceed. Trivial implementation details don't need a round-trip.
3. Follow the conventions already in the files/modules you touch.
4. Mark a task `- [x]` in `plan.md` as soon as its done-criteria is met — keeps progress resumable if the run stops early.
5. Necessary work surfaces outside this phase's task list → don't do it silently. Note it in `follow-up.md` (create from `.claude/templates/follow-up.md` if needed, or append) and flag it to the user.

## Phase 5 — Verify

1. Detect the project's build/lint/test commands: `CLAUDE.md` first, then `package.json` scripts / `Makefile` / equivalent. Still unclear → ask the user once rather than guessing at commands that might not exist.
2. Run them. Failures → fix and re-run.
3. A failure exposes a gap in the plan itself, not just a bug in the new code → surface it to the user rather than silently deviating from the approved plan.
4. UI-facing changes → exercise the feature in a running app if you're able to; otherwise tell the user this still needs manual verification.

## Phase 6 — Approval loop

1. Summarize the phase's changes — files touched, diff stat, verification results.
2. Approve or request changes, loop until approved.
3. Approval is a chat-only gate for Phase 7 — never add an approver name to any file.

## Phase 7 — Commit, push, PR

**Don't start until Phase 6 approval is explicit.**

1. Stage the changed application files plus `plan.md` (and `follow-up.md` if updated).
2. Commit. Default to one commit for the phase; split further only if the tasks are genuinely separable and the user prefers that. Message: `Implement <phase title> for <ticketId>`.
3. Push the phase branch, setting upstream if it has none.
4. Check `plan.md` for unchecked tasks in any _other_ phase. None remain → this is the last phase; the PR body must include a `- [ ] QA Done` checkbox (unchecked, ticked by `/codely.qa` once QA passes, gates the merge — QA runs once for the whole plan, not per phase). Other phases still have unchecked tasks → omit the checkbox entirely from this PR's body.
5. This is the last phase (per step 4) and `follow-up.md` exists with any Open or pending items → the PR body must also include a `- [ ] Follow-Up Triaged` checkbox (unchecked, ticked by `/codely.follow-up` once every item is resolved). Not the last phase, or no `follow-up.md`/items yet → omit it; `/codely.follow-up` adds it later if needed.
6. `gh` available → `gh pr create --draft` (base = the branch from Phase 2, head = the phase branch, title referencing ticket + phase, body linking back to the plan's phase section, plus the `QA Done` and/or `Follow-Up Triaged` checkboxes per steps 4-5 if applicable). `gh` unavailable → tell the user to push and open the PR manually, including the checkboxes per steps 4-5 if applicable.
7. Confirm: files changed, commit(s), PR URL (or manual-PR instructions), and which gate checkboxes this PR carries.

## Phase 8 — Next steps

1. `plan.md` still has unchecked tasks in other phases → tell the user to get this PR reviewed/merged, then run `/clear` then `/codely.implement` again for the next phase.
2. All phases now fully checked → tell the user to run `/clear` then `/codely.qa` next, which is responsible for marking the PR ready for review once it passes.
