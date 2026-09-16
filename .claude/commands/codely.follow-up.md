---
description: Triage a ticket's follow-up items one at a time — promote, integrate, drop, or defer to the user
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.follow-up

> **Scope guardrail:** triages `follow-up.md` items one at a time — Promote (ticket creation, via a connected task-tracking MCP tool if available, else a manual draft), Integrate (small changes folded into the current PR), Drop (with a mandatory reason), or Other (user's call). Within `.specs/<ticketId>_<slug>/`, touches only `follow-up.md`. Outside it, when `.sdlc/backlog.md` is the ticket source, may add promoted follow-up tickets to it and set this ticket's Status to `done`. Application code changes happen only for items dispositioned Integrate. Outputs are the updated `follow-up.md`, any Integrate code changes, and — with no separate approval step, since every disposition was already the user's decision — a commit + push + PR comment, with `Follow-Up Triaged` ticked once every item is resolved.

Triage `.specs/<ticketId>_<slug>/follow-up.md` one item at a time. Follow the phases in order. Git changes happen only in Phase 6, after every item has a decision and any Integrate work passes verification. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

`follow-up.md` holds items deferred during interviews or surfaced during implementation, one `### N. {{item_title}}` entry per item with a `**Surfaced:**`/`**Description:**`/`**Status:**` block — triaged one at a time by this command.

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve the ticket and follow-up file

1. Read `CLAUDE.md` if present.
2. Determine `ticketId`: current branch matches a phase branch `<ticketId>/<phaseSlug>`, a quick branch `<ticketId>/<slug>/quick`, a bug-fix branch `<ticketId>/<slug>/bug-fix`, or a docs branch `<ticketId>/<slug>/feature` → extract it. Trunk, or neither pattern matches → use `$ARGUMENTS` if given, else — inside a ticket worktree (see `.claude/includes/worktree-ticket.md`) — the worktree's ticket ID, else ask. Don't guess.
3. Locate `.specs/<ticketId>_*`. Missing → stop, tell the user there's nothing to triage for this ticket.
4. `.specs/<ticketId>_<slug>/follow-up.md` missing → tell the user there are no follow-up items for this ticket, stop.
5. Parse each item's Status: Open / Promoted (pending ticket ID) / Triaged.
6. Nothing Open or pending → tell the user everything is already triaged, stop.

## Phase 1 — Load context

1. Read `spec.md` (if present) and the plan document (`plan.md`, `quick-plan.md`, or `bug-fix-plan.md`) — needed to draft good ticket descriptions and Drop-reason suggestions.
2. Note where new tickets go: `.claude/includes/task-tracker.md` says `Not used for this repo` → `.sdlc/backlog.md` is the tracker. Otherwise note whether a task-tracking MCP connector (Plane, Jira/Atlassian, Linear, Asana, monday.com, etc.) is available and authorized in this session.

## Phase 2 — Resolve pending promotions

For any item already dispositioned Promote but still awaiting a ticket ID — **one item at a time**, same numbering style as Phase 3 (`Pending item <k> of <P>`):

1. Remind the user in one or two plain sentences what the item is, then AskUserQuestion (one question) whether the ticket has been created yet.
2. Yes → ask for the ticket ID/URL right away, record it, set Status to Triaged, save `follow-up.md`.
3. Still not → leave it as-is; it'll come up again next time `/codely.follow-up` runs.
4. Confirm in one line, then move to the next pending item.

## Phase 3 — Triage the remaining items one by one

**One item at a time — never batch.** Number the Open items `1..M` in file order. Before the first, tell the user in one line how many items need a decision. Each item is fully finished — decision, every follow-up question that decision needs, and the file update — before the next item is shown.

For each Open item, in order:

1. Post a message laid out like this, in plain language — no jargon or file paths unless the user needs them to decide:

   ```
   **Item <k> of <M>: <item title>**

   **What it is:** <one or two sentences on the problem or idea>

   **Where it came from:** <when/why it surfaced — interview, implementation, review, QA>

   **My recommendation:** <Promote / Integrate / Drop, and why in one sentence>
   ```

2. AskUserQuestion — one question only, header `Item <k>/<M>`, "What should happen with this item?", recommended option first and marked `(Recommended)`:
   - **Promote** — create a ticket for it.
   - **Integrate** — small enough to fold into this PR right now.
   - **Drop** — skip it entirely, with a reason.
   - Anything else → the user types it in the free-text "Other" answer.
3. **Immediately ask whatever that decision needs — before moving to the next item:**
   - **Promote:**
     - `.sdlc/backlog.md` is the tracker (Phase 1) → show a drafted title + one-line description and AskUserQuestion — add it as drafted / let me edit it first. Then append a row to the backlog's last wave table (or a `## Follow-ups` wave, created if missing) with the next free ticket ID per `.claude/includes/ticket-format.md`, Source `follow-up of <ticketId>`, Depends On `<ticketId>` if it builds on this work, a size estimate, and Status `to do`. Record the new ticket ID, Status: Triaged. Stage `.sdlc/backlog.md` with Phase 6's commit.
     - Otherwise, a task-tracking MCP connector (Plane, Jira/Atlassian, Linear, Asana, monday.com, etc.) is available and authorized → show a drafted title + description from the item and AskUserQuestion — create it as drafted / let me edit it first. Edit → take their changes, then create it. Record the resulting ticket ID/URL, Status: Triaged.
     - No connector available, or the user prefers manual → show a drafted title + description for them to paste into their tracker, then ask for the resulting ticket ID. Provided now → record it, Status: Triaged. Not yet → Status: "Promoted (pending ticket ID)", resolved by a future run's Phase 2.
   - **Integrate:** confirm in one sentence what the change will be, and ask about anything material the item's description leaves open. Record the intent now; the actual implementation happens in Phase 4. Don't mark Triaged until it's actually done and verified.
   - **Drop:** propose a reason grounded in the item's context; AskUserQuestion — accept the suggested reason / write my own (free text). Record it, Status: Triaged.
   - **Other (free text):** their answer is unclear or incomplete → ask one clarifying question. Record what they decided as the disposition, Status: Triaged.
4. Save `follow-up.md` right away — don't batch updates to the end: append a `**Disposition:**` line directly below `**Status:**`, e.g. `**Disposition:** Promote — PROJ-456 created 2026-07-20`, `**Disposition:** Integrate — implemented in this PR`, `**Disposition:** Drop — reason`, or `**Disposition:** Other — what was decided`.
5. Confirm in one line (e.g. `Item 2 of 5: Dropped — covered by PROJ-41`), then move to the next item.

The user asks a question mid-item → answer it, then re-ask the same item's pending question; don't advance until the item is fully recorded.

## Phase 4 — Integrate the chosen items

1. No items dispositioned Integrate → skip to Phase 4b.
2. For each: explore the relevant files, implement the change, follow existing conventions. A material decision isn't resolved by the item's description → don't guess: AskUserQuestion, then proceed.
3. Verify: run the project's build/lint/test commands (`CLAUDE.md` first, then `package.json`/`Makefile`/equivalent). Failures → fix and re-run.
4. Mark each Integrate item's Status Triaged once implemented and verified.
5. Keep a list of every file each Integrate item changed — code, tests, config, docs — for Phase 6.

## Phase 4b — Mark the backlog ticket done

@.claude/includes/ticket-source.md

Completion condition for this command — both must hold: every item in `follow-up.md` is now Triaged (none Open or pending); and QA already passed — the plan document's QA Plan has every item either checked or marked Deferred, none Failed or unrecorded — and Phase 4 changed no application code (an Integrate change makes the earlier QA results stale). Met → set the backlog row to `done`. QA not run, not passed, or stale → leave it and tell the user to re-run `/codely.qa`, which will mark it done.

## Phase 5 — Summary

1. Post a short summary: every item's disposition, plus any Integrate code changes (files touched, diff stat, verification results). This is informational — don't ask for approval; go straight on to Phase 6.
2. Integrate verification is still failing after your fixes (Phase 4 step 3) → **stop here**: don't commit, don't push, don't touch the PR. Report which check failed and what you tried.

## Phase 6 — Commit, push, tick Follow-Up Triaged

Runs directly after Phase 5 — no approval gate.

1. Build this run's file list: `follow-up.md`, every file on Phase 4's Integrate list, and `.sdlc/backlog.md` if Phase 3 or 4b changed it. Stage exactly those files. None of them changed (e.g. only pending promotions left unchanged) → skip steps 2–4, tell the user which items are still pending, and go to step 6.
2. Commit only that file list, so anything the user had staged before this run stays out of the commit: `git commit -m "Triage follow-up items for <ticketId>" -- <files>`.
3. Push the current branch.
4. `gh` available → find the open PR for this branch, fetch its current body. No `Follow-Up Triaged` checkbox present → add `- [ ] Follow-Up Triaged` to it. Every item now Triaged (none Open or pending) → set it to `- [x] Follow-Up Triaged`; otherwise set it to `- [ ] Follow-Up Triaged`, even if it was ticked before. Integrate items changed application code and `- [x] QA Done` is ticked → flip it back to `- [ ] QA Done` and tell the user to re-run `/codely.qa`. Either way, write the updated body back via the REST API: `gh api repos/{owner}/{repo}/pulls/<number> -X PATCH -f body=<updated body>` (not `gh pr edit --body`, which drives a GraphQL mutation that can fail with a `Projects (classic)` deprecation error on repos with classic project boards). Add a PR comment summarizing dispositions (Promoted/Integrated/Dropped/Other counts, with ticket links for promoted items). `gh` unavailable → tell the user to tick it manually and share the summary. If Integrate items changed application code, also tell them to untick `QA Done` if it's ticked and re-run `/codely.qa`.
5. Items still pending (promotions awaiting a ticket ID) → the checkbox stays unticked (step 4 unticks it if needed), tell the user why, and that re-running `/codely.follow-up` later will finish the job.
6. Confirm: `follow-up.md` path, PR URL, summary.

## Phase 7 — Next steps

1. Any promoted tickets → remind the user to track them separately.
2. Any Integrate changes → note they're now part of this PR.
3. Tell the user to get the PR reviewed and merged once `QA Done` and `Follow-Up Triaged` (where applicable) are both checked. Anything in this run's commit they want changed → they can say so and it'll go in a new commit.
