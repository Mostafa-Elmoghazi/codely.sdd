---
description: Turn the approved system design into a proposed, board-agnostic ticket backlog
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
disable-model-invocation: true
---

# codely.backlog

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/backlog.md`. This command makes no git changes and never creates tickets on any real dev board — its output is a proposal for a human to turn into real tickets on whatever tracker this project uses (Jira/Linear/GitHub Issues/etc.).

Turn the approved SDLC design into a proposed ticket backlog at `.sdlc/backlog.md` — one entry per proposed ticket, each traceable back to the SRS's functional requirements and the LLD's components/endpoints, grouped into dependency-ordered waves. This is the bridge between the SDLC design phases and the ticket-by-ticket `codely.*` implementation loop: once a ticket from this backlog is created on your board with a real ID, run `/codely.spec <ticketId> <description>` to start it. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

## Phase 0 — Load the design

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md`. **Missing either:** tell the user backlog generation needs an approved system design first, stop, recommend `/sdlc.system-design`.
3. Read both, plus `.sdlc/srs.md` and `.sdlc/use-case-model.md` — functional/non-functional requirements and use cases/actors give the "why" behind each proposed ticket; components and API endpoints give the "what."
4. Glob `.sdlc/project-plan.md` — present → read it too (its roadmap's phase sequencing can inform how tickets get grouped into waves).
5. `$ARGUMENTS` non-empty: treat it as extra focus for this pass (e.g. "just the auth module" to extend an existing backlog incrementally).

## Phase 1 — Check for an existing backlog

1. Glob `.sdlc/backlog.md`.
2. **Missing:** proceed to Phase 2.
3. **Found:** read and summarize it — existing tickets, which waves, which already show a Board ID. AskUserQuestion — Refresh (add tickets for anything new in the design since this was last generated, keep existing entries and their Board IDs untouched) / Start over (regenerate entirely) / Cancel.

## Phase 2 — Derive candidate tickets

1. Cross-reference the SRS's Functional Requirements with the LLD's Component Design and API Endpoints — group related requirements by the component(s) that implement them.
2. Fold in the HLD's UI/UX Overview (if present) so UI-only work isn't missed.
3. For each candidate ticket, work out: a title, a one-line description, which FR(s)/component(s)/use case(s) it covers, a rough size (S/M/L, based on files/components touched), and what other candidate tickets it depends on.
4. Group candidates into dependency-ordered waves (a ticket goes in the first wave where all its dependencies are satisfied) — align with the project plan's roadmap phases where one exists, but dependencies win over any mismatch.

## Phase 3 — Review with the user (rounds)

**Mandatory — don't write the backlog until the user has confirmed the slicing, sizing, and sequencing.**

1. Present the candidate tickets grouped by wave. AskUserQuestion — confirm as-is, or adjust (split, merge, reprioritize, add missing, drop out-of-scope work) — cap 4 options/call, multiple rounds if needed.
2. Digest each round; iterate until the user is satisfied with the slicing.
3. On "you decide" calls (sizing ties, sequencing calls), state a recommendation and get explicit confirmation — record it under Decisions.

## Phase 4 — Write the backlog

1. Fill `.claude/templates/backlog.md` placeholders — the template holds only headers, the "How to use this" note, and placeholder tokens, so use the guidance below for what belongs in each section; don't carry this guidance itself into the written file.
2. Mandatory: at least one Wave table — repeat `## Wave N: {{title}}` with its own table for each additional wave (Proposed Title, Description, Source, Depends On, Size — leave Board ID blank). Tickets within a wave have no dependencies on each other; later waves depend on earlier ones.
3. Optional — omit the whole section if empty, never write "N/A": Out of Scope / Deferred (candidate work identified but deliberately excluded from this backlog — e.g. explicitly out of scope per the charter, or deferred to a later phase of the project plan), Decisions ("you decide" calls made during the review, with rationale — sizing ties, sequencing calls, slicing choices).
4. Write to `.sdlc/backlog.md`.

## Phase 5 — Approval loop & handoff

1. Ask the user to review `.sdlc/backlog.md` — approve or request changes, loop until approved.
2. No git action follows approval — this command never branches, commits, or pushes, and never touches any external tracker.
3. Confirm the artifact path. Tell the user: create each ticket on your team's actual dev board from this list (recording the real ID back into the Board ID column is optional, for your own reference only), then run `/clear` then `/codely.spec <ticketId> <description>` per ticket, in the wave order shown, to start implementation.
