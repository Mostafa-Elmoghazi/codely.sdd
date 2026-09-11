---
description: Verify plan completion and walk the QA Plan against the implementation, then tick QA Done and mark the PR ready for review
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.qa

> **Scope guardrail:** verifies, never fixes — if a QA item fails, this command stops and reports rather than modifying application code. Within `.specs/<ticketId>_<slug>/`, touches only the plan document (`plan.md` or `quick-plan.md`) to record QA Plan results — never `spec.md`, never application code. Outputs are the QA results recorded in the plan document and — once every item is Passed or Deferred, with none Failed — a commit + push + a ticked `QA Done` checkbox on the PR + the PR marked ready for review.

Verify every task across the plan is complete, then work through the plan's QA Plan checklist against the implementation — running what can be verified programmatically without asking, and walking everything else with the user one item at a time. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve the ticket and plan document

1. Read `CLAUDE.md` if present.
2. Determine `ticketId`: current branch matches a phase branch `<ticketId>/<phaseSlug>`, a quick branch `<ticketId>/<slug>/quick`, a bug-fix branch `<ticketId>/<slug>/bug-fix`, or a docs branch `<ticketId>/<slug>/feature` → extract it. Trunk, or neither pattern matches → use `$ARGUMENTS` if given, else ask. Don't guess.
3. Locate `.specs/<ticketId>_*`. Missing → stop, recommend `/codely.spec`, `/codely.quick`, or `/codely.bug-fix`.
4. The directory has `quick-plan.md` → that's the plan document for this run. Else it has `bug-fix-plan.md` → that's the plan document (its Verification section's checklist items are this run's QA Plan). Else it has `plan.md` → that's the plan document. Neither → stop, tell the user to run `/codely.plan`, `/codely.quick`, or `/codely.bug-fix` first.
5. Parse every task across every phase (or the single phase, for `quick-plan.md`). Any unchecked → tell the user which, AskUserQuestion — stop and finish implementation first (recommend `/codely.implement`) / proceed with QA anyway (confirm — unusual).
6. Current branch isn't the branch those tasks were implemented on (the last unmerged phase branch, or the quick branch) → warn, ask whether to check it out or continue anyway.

## Phase 1 — Load context

1. `spec.md` present (full flow) → read it.
2. `system-design.md` present → read it.
3. `design.md` present → read it, to verify the implementation matches the approved screens.
4. Read the plan document in full, including its QA Plan section.
5. `follow-up.md` present → read it, for awareness only.

## Phase 2 — Classify QA Plan items

For each QA Plan checklist item, decide: machine-verifiable (a concrete, runnable check — e.g. "all unit tests pass," "build succeeds," "lint is clean," a specific behavior scriptable via a command or targeted grep/read) vs. needs human judgment (visual/UX, business-logic nuance, anything you can't conclusively check yourself). When genuinely unsure, treat it as needing human judgment — don't guess a pass.

Plan document is `bug-fix-plan.md` → also confirm its Verification → Regression Test evidence (commit references plus before/after output) is present and internally consistent before walking the rest of the checklist. Treat a missing or inconsistent record as a failed item, not something to wave through.

## Phase 3 — Run the machine-verifiable items

1. For each item classified as machine-verifiable, run the check (tests, build, lint, targeted script, etc.).
2. Record Passed or Failed against the item, with a short note (command run / relevant output) — no need to ask the user.
3. Report each result to the user as you go, not only in a final summary.

## Phase 4 — Walk the remaining items with the user

For each item classified as needing human judgment (batch up to 4 per AskUserQuestion call):

1. Explain what the item verifies, in plain terms.
2. AskUserQuestion with options: Passed / Failed / Defer (post-deploy manual check).
3. **Failed** → ask a brief follow-up for what failed / repro steps; record it against the item.
4. **Defer** → ask for the note to attach (why it's deferred, what to check post-deploy); record it against the item.

## Phase 5 — Update the plan document

1. Passed → `- [x] {{test_case}}`.
2. Failed → leave unchecked, append `— **Failed**: <details>`.
3. Deferred → leave unchecked, append `— **Deferred (post-deploy manual check)**: <note>`.
4. Save the plan document.

## Phase 6 — Gate on results

1. Any item Failed → tell the user QA did not pass, list the failures, recommend fixing them (via `/codely.implement` or directly) and re-running `/codely.qa`. Stop here — don't push, don't touch the PR, don't tick `QA Done`.
2. No Failed items (Deferred items are fine) → continue.

## Phase 7 — Commit, push, tick QA Done, mark the PR ready

**Don't start until Phase 6 clears with no failures.**

1. `git status --porcelain`, disregarding the plan document just edited in Phase 5 — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort).
2. Stage and commit the plan document: `Record QA results for <ticketId>`.
3. Push the current branch.
4. `gh` available → find the open PR for this branch, fetch its current body, flip `- [ ] QA Done` to `- [x] QA Done` within it, and write the updated body back via the REST API: `gh api repos/{owner}/{repo}/pulls/<number> -X PATCH -f body=<updated body>` (not `gh pr edit --body`, which drives a GraphQL mutation that can fail with a `Projects (classic)` deprecation error on repos with classic project boards). This is the checkbox that gates the merge — tick it only now that Phase 6 has cleared.
5. Add a QA summary comment (`gh pr comment`) — Passed/Failed/Deferred counts, deferred items listed explicitly.
6. `gh pr ready` to remove draft status.
7. `gh` unavailable → tell the user to tick the `QA Done` checkbox and mark the PR ready for review manually, and share the QA summary for them to paste in.
8. Confirm: plan document path, PR URL, QA summary.

## Phase 8 — Next steps

1. Any Deferred items → remind the user they need manual verification post-deploy; point back at the plan document's QA Plan section for the list.
2. `.specs/<ticketId>_<slug>/follow-up.md` exists → tell the user to run `/clear` then `/codely.follow-up` next.
3. No `follow-up.md` → tell the user to get the (now ready-for-review, `QA Done`-checked) PR reviewed and merged.
