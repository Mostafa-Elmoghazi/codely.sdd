---
description: Reproduce a reported bug, confirm its root cause, plan and implement a fix proven by a regression test, and open a draft PR
argument-hint: [ticket-id] [brief description]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.bug-fix

> **Scope guardrail:** for bug fixes only — root cause must be confirmed with the user before any fix code is written, and the fix must be proven by a regression test shown failing before the fix and passing after (each in its own commit). Within `.specs/<ticketId>_<slug>/`, writes only `bug-fix-plan.md` and, if deferred items surface, `follow-up.md`. Two chat-only approval gates — plan, then implementation — stand before any git-mutating step. Outputs are the bug-fix plan, a red-then-green regression test, the application code for the fix, and — after both approvals — a commit history, push, and draft PR on one new branch.

Turn a ticket ID + bug report into a reproduced, root-caused, reviewed, working fix — investigate, confirm root cause, write a single `bug-fix-plan.md`, implement behind a failing→passing regression test, and open a draft PR, all in one run. Follow the phases in order. No git changes before the relevant approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`command -v gh >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve inputs

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: first token (minus trailing `:`) = ticket ID, rest = bug description.
3. `$ARGUMENTS` empty: ask the user for both. Don't continue without them.

## Phase 1 — Load the ticket ID format

1. Read `.claude/includes/ticket-format.md`.
2. **Missing** (first run): tell the user, then AskUserQuestion — "describe the format, I'll draft the file" vs. "I'll set it up myself, stop here." If they describe it, ask the scheme (JIRA/GitHub/Linear/none/other), write the file, continue. Otherwise stop.
3. **Present:** extract the pattern/regex and examples.
4. Validate the candidate ticket ID against it.
5. **No match** (or no candidate): show the expected format, AskUserQuestion — corrected ID / use anyway / no-ticket fallback. Never guess.

## Phase 2 — Check for an existing ticket

1. Glob `.specs/` for `<ticketId>_*`. None → go to Phase 3.
2. Found, with `spec.md`, `plan.md`, or `quick-plan.md` but no `bug-fix-plan.md` (this ticket ID is already something else) → tell the user, AskUserQuestion — proceed with bug-fix anyway (confirm — unusual) / cancel.
3. Found, with `bug-fix-plan.md` → read + summarize it, AskUserQuestion — Resume/revise (reuse slug, skip reproduction/investigation) / Start over (reuse slug, redo reproduction/investigation) / Cancel. Either way skip Phase 3 and reuse the existing slug.

## Phase 3 — Generate the slug (new tickets only)

From the ticket title: lowercase, `-`/`_` only, no dots, ~5 words / 40 chars max. Generate once, reuse everywhere below.

## Phase 4 — Reproduce the bug

1. From `$ARGUMENTS`'s description and anything else the user has already provided (repro steps, logs, screenshots), attempt to reproduce the bug locally — run the app or the relevant tests, follow the described steps.
2. Reproducible → capture the exact failure verbatim (error message, stack trace, failing assertion, or the specific incorrect output observed).
3. Not reproducible from what's given → AskUserQuestion for what's missing: exact steps, environment (OS/browser/version/deployment), account/data state, frequency (always/intermittent), when it started, logs or screenshots. Retry with the additional detail.
4. Still not reproducible after that round → tell the user, AskUserQuestion — continue investigating from code inspection alone (lower confidence; flag this in Root Cause) / stop here so the user can gather more repro information first. Never fabricate a reproduction.

## Phase 5 — Investigate

Read-only exploration — don't ask the user what the code and its history already answer.

1. Trace the failure from symptom back to source: Glob/Grep/Read the implicated code path.
2. `git log`/`git blame` the relevant files — is this a regression (find the introducing commit) or a longstanding defect?
3. Check whether the same defect pattern exists elsewhere in the codebase (duplicated logic, similar components). Note it, but don't fix it here — it goes to `follow-up.md` (Phase 8).
4. Form a specific, falsifiable root-cause hypothesis: the exact line(s)/logic responsible, and why they produce the observed symptom.

## Phase 6 — Confirm the root cause

**Mandatory gate — don't draft the Fix Plan until this is confirmed.** Coding against a wrong hypothesis costs more than one extra round-trip.

1. Present the hypothesis and its evidence (code path, git history, Phase 4's reproduction output) in plain terms.
2. AskUserQuestion — Confirmed, proceed / Not quite — here's what's off (record the correction, loop back to Phase 5) / Not sure, use your best judgment (state the hypothesis you'll proceed with and get explicit confirmation of that fallback).
3. Reproduction couldn't distinguish between multiple plausible root causes → say so; ask the user to help disambiguate rather than guessing.

## Phase 7 — Draft the Fix Plan

1. Propose a fix approach that addresses the confirmed root cause, not just the symptom. Genuine alternative approaches exist → AskUserQuestion with trade-offs; on "you decide," state a recommendation and get explicit confirmation.
2. Define Expected Behavior concretely enough to become the regression test's assertion.
3. Break the fix into Tasks. Task 1 is always the regression test; the rest implement the fix.
4. Note Key Files.
5. Based on Phase 5's investigation, decide whether an automated regression test is feasible for this bug — most bugs are; note if this one genuinely isn't (e.g. purely visual/environment-specific) so Phase 12 knows to fall back to Manual Verification.

## Phase 8 — Deferred items

1. Phase 5 or 7 surfaced related defects or points out of scope for this fix → tell the user they'll be captured for later triage rather than folded into this fix.
2. Create (or append to) `.specs/<ticketId>_<slug>/follow-up.md` from `.claude/templates/follow-up.md` with those items.
3. These get triaged one at a time later via `/codely.follow-up`.

## Phase 9 — Write the bug-fix plan

1. Fill `.claude/templates/bug-fix-plan.md` placeholders.
2. Mandatory: Problem, Root Cause, Expected Behavior, Fix Plan, Tasks, Key Files, Verification (Regression Test subsection left as a skeleton — real evidence is filled in during Phases 12-13).
3. Optional — omit the whole section if empty, never write "N/A": Out of Scope, Manual Verification (if the Regression Test alone fully covers it).
4. Branch: `<ticketId>/<slug>/bug-fix`, exact order.
5. Write to `.specs/<ticketId>_<slug>/bug-fix-plan.md`, creating the directory if needed.

## Phase 10 — Plan approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/bug-fix-plan.md` (and `follow-up.md` if created) — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 11 — never add an approver name to the file.

## Phase 11 — Branch creation & plan commit

**Don't start until Phase 10 approval is explicit.**

1. `git status --porcelain`, disregarding `bug-fix-plan.md`/`follow-up.md` just written in Phase 9 — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Trunk: `CLAUDE.md`'s choice, else `main`, else `master`. Ask if still ambiguous.
3. Zero commits (`git rev-parse --verify HEAD` fails): tell the user, skip switch/pull, ask whether to branch from the unborn state or commit first.
4. Otherwise: switch to trunk if needed; pull only if it tracks a remote.
5. Create/checkout `<ticketId>/<slug>/bug-fix`. If it already exists, check it out and note how far ahead of trunk it is.
6. Stage and commit `bug-fix-plan.md` (and `follow-up.md` if created): `Add bug-fix plan for <ticketId>`.
7. Don't push yet — implementation is still pending.

## Phase 12 — Regression test (red)

The proof that this fix works starts here, before any fix code is touched.

1. Write a test encoding Expected Behavior, placed and named per the codebase's existing test conventions.
2. Run it in isolation. It must fail. Confirm the failure reason matches the confirmed root cause, not a setup error or typo in the test itself — if it fails for the wrong reason, fix the test and re-run.
3. Genuinely not testable in an automated way (per Phase 7's note, or discovered only now) → mark the plan's Regression Test subsection "Not applicable — <reason>," tell the user, and rely on Manual Verification instead (Phase 14). Skip to Phase 13.
4. Capture the failing output verbatim.
5. Stage and commit *only* the test: `Add regression test for <ticketId> (reproduces bug, currently failing)`. This commit is the durable, independently-checkable proof that the test fails without the fix.
6. Record the test reference, this commit's SHA, and the failing output in `bug-fix-plan.md`'s Verification → Regression Test → "Before fix (red)".

## Phase 13 — Implement the fix (green)

1. Work through the remaining unchecked Tasks from `bug-fix-plan.md`, in order.
2. Follow the conventions already in the files/modules you touch.
3. Mark each task `- [x]` as its done-criteria is met.
4. Necessary work surfaces outside these tasks → don't do it silently. Note it in `follow-up.md` (create from `.claude/templates/follow-up.md` if needed, or append) and flag it to the user.
5. Regression test was written in Phase 12 → re-run it now. It must pass. Doesn't pass → keep fixing; never weaken the test to force a pass.
6. Capture the passing output verbatim. Record it in Verification → Regression Test → "After fix (green)", referencing the fix commit by the exact message it will be committed with in Phase 16 (that commit doesn't exist yet — it's identified by message, not SHA).

## Phase 14 — Verify

1. Detect the project's build/lint/test commands: `CLAUDE.md` first, then `package.json` scripts / `Makefile` / equivalent. Still unclear → ask the user once rather than guessing at commands that might not exist.
2. Run the full suite, not just the new regression test — confirms no regressions elsewhere. Failures unrelated to this fix → surface to the user rather than silently patching around them.
3. Re-walk the Steps to Reproduce from Problem against the fixed code — confirm the original symptom no longer occurs. Record the result under Manual Verification (if Phase 12 fell back to "not applicable," this is now the primary evidence — be thorough).
4. UI-facing bug → exercise it in a running app if you're able to; otherwise tell the user this still needs manual verification.
5. Tick the Verification checklist items (Manual Verification steps, Regression Check) in `bug-fix-plan.md` accordingly.

## Phase 15 — Implementation approval loop

1. Summarize the fix — files touched, diff stat, the regression test's before/after output, verification results.
2. Approve or request changes, loop until approved.
3. Approval is a chat-only gate for Phase 16 — never add an approver name to any file.

## Phase 16 — Commit, push, draft PR

**Don't start until Phase 15 approval is explicit.**

1. Stage the changed application files plus `bug-fix-plan.md` (task checkboxes, Verification updates). The regression test was already committed separately in Phase 12 — don't re-stage it here unless it changed since.
2. Commit: `Fix <ticketId>: <title>` — the exact message referenced in the plan doc's Verification → Regression Test → "After fix (green)".
3. Push the branch, setting upstream if it has none.
4. `follow-up.md` exists with any Open or pending items → the PR body must also include a `- [ ] Follow-Up Triaged` checkbox. No `follow-up.md`, or no items yet → omit it.
5. Trunk = `CLAUDE.md`'s choice, else `main`, else `master`. `gh` available → `gh pr create --draft` (base = trunk, head = `<ticketId>/<slug>/bug-fix`, title referencing the ticket, body summarizing Problem/Root Cause/Fix from `bug-fix-plan.md`, noting the two-commit red→green proof with the regression-test commit's SHA, and including a `- [ ] QA Done` checkbox plus the `- [ ] Follow-Up Triaged` checkbox per step 4 if applicable). `gh` unavailable → tell the user to push and open the PR manually, including the same checkboxes.
6. Confirm: plan path, files changed, both commits (test SHA + fix commit), PR URL (or manual-PR instructions), and which gate checkboxes this PR carries.

## Phase 17 — Next steps

1. Tell the user to get the PR reviewed and merged.
2. Mention `/codely.qa` as an optional final verification pass before or after merge, then `/clear`.
