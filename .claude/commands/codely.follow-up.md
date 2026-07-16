---
description: Triage a ticket's follow-up items one at a time — promote, integrate, drop, or defer to the user
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.follow-up

> **Scope guardrail:** triages `follow-up.md` items one at a time — Promote (ticket creation, via a connected task-tracking MCP tool if available, else a manual draft), Integrate (small changes folded into the current PR), Drop (with a mandatory reason), or Other (user's call). Within `.specs/<ticketId>_<slug>/`, touches only `follow-up.md`. Application code changes happen only for items dispositioned Integrate, and only after approval. Outputs are the updated `follow-up.md`, any Integrate code changes, and — after approval — a commit + push + PR comment, with `Follow-Up Triaged` ticked once every item is resolved.

Triage `.specs/<ticketId>_<slug>/follow-up.md` one item at a time. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- `gh` CLI available: !`command -v gh >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Resolve the ticket and follow-up file

1. Read `CLAUDE.md` if present.
2. Determine `ticketId`: current branch matches a phase branch `<ticketId>/<phaseSlug>`, a quick branch `<ticketId>/<slug>/quick`, or a docs branch `<ticketId>/<slug>/feature` → extract it. Trunk, or neither pattern matches → use `$ARGUMENTS` if given, else ask. Don't guess.
3. Locate `.specs/<ticketId>_*`. Missing → stop, tell the user there's nothing to triage for this ticket.
4. `.specs/<ticketId>_<slug>/follow-up.md` missing → tell the user there are no follow-up items for this ticket, stop.
5. Parse each item's Status: Open / Promoted (pending ticket ID) / Triaged.
6. Nothing Open or pending → tell the user everything is already triaged, stop.

## Phase 1 — Load context

1. Read `spec.md` (if present) and the plan document (`plan.md` or `quick-plan.md`) — needed to draft good ticket descriptions and Drop-reason suggestions.
2. Note whether a task-tracking MCP connector (Jira/Atlassian, Linear, Asana, monday.com, etc.) is available and authorized in this session.

## Phase 2 — Resolve pending promotions

For any item already dispositioned Promote but still awaiting a ticket ID:

1. Ask the user whether the ticket has been created yet.
2. Yes → record the ticket ID/URL, set Status to Triaged.
3. Still not → leave it as-is; it'll come up again next time `/codely.follow-up` runs.

## Phase 3 — Triage the remaining items one by one

For each Open item:

1. Present the item's title, description, and where it was surfaced.
2. AskUserQuestion — "What should happen with this item?":
   - **Promote** — create a ticket for it.
   - **Integrate** — small enough to fold into this PR right now.
   - **Drop** — skip it entirely, with a reason.
   - **Other** — something else; the user decides.
3. **Promote:**
   - A task-tracking MCP connector is available and authorized → offer to create the ticket now (draft title + description from the item, let the user confirm or edit, then create it). Record the resulting ticket ID/URL, Status: Triaged.
   - No connector available, or the user prefers manual → draft a title + description for them to paste into their tracker, then ask them to reply with the resulting ticket ID. Provided now → record it, Status: Triaged. Not yet → Status: "Promoted (pending ticket ID)", resolved by a future run's Phase 2.
4. **Integrate:** record the intent now; the actual implementation happens in Phase 4. Don't mark Triaged until it's actually done and verified.
5. **Drop:** propose a reason grounded in the item's context; AskUserQuestion — accept the suggested reason / provide your own. Record it, Status: Triaged.
6. **Other:** ask the user what they want done; record their answer verbatim as the disposition, Status: Triaged.
7. Update the item in `follow-up.md` immediately after each decision — don't batch updates to the end.

## Phase 4 — Integrate the chosen items

1. No items dispositioned Integrate → skip to Phase 5.
2. For each: explore the relevant files, implement the change, follow existing conventions. A material decision isn't resolved by the item's description → don't guess: AskUserQuestion, then proceed.
3. Verify: run the project's build/lint/test commands (`CLAUDE.md` first, then `package.json`/`Makefile`/equivalent). Failures → fix and re-run.
4. Mark each Integrate item's Status Triaged once implemented and verified.

## Phase 5 — Approval loop

1. Summarize: every item's disposition, plus any Integrate code changes (files touched, diff stat, verification results).
2. Approve or request changes, loop until approved.
3. Approval is a chat-only gate for Phase 6 — never add an approver name to any file.

## Phase 6 — Commit, push, tick Follow-Up Triaged

**Don't start until Phase 5 approval is explicit.**

1. Stage `follow-up.md` plus any application files changed by Integrate items.
2. Commit: `Triage follow-up items for <ticketId>`.
3. Push the current branch.
4. `gh` available → find the open PR for this branch. Its body has no `Follow-Up Triaged` checkbox → add `- [ ] Follow-Up Triaged` first. Every item now Triaged (none Open or pending) → flip it to `- [x] Follow-Up Triaged` via `gh pr edit --body`. Add a PR comment summarizing dispositions (Promoted/Integrated/Dropped/Other counts, with ticket links for promoted items). `gh` unavailable → tell the user to tick it manually and share the summary.
5. Items still pending (promotions awaiting a ticket ID) → leave the checkbox unticked, tell the user why, and that re-running `/codely.follow-up` later will finish the job.
6. Confirm: `follow-up.md` path, PR URL, summary.

## Phase 7 — Next steps

1. Any promoted tickets → remind the user to track them separately.
2. Any Integrate changes → note they're now part of this PR.
3. Tell the user to get the PR reviewed and merged once `QA Done` and `Follow-Up Triaged` (where applicable) are both checked.
