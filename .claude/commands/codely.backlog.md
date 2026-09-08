---
description: Turn the approved system design into a proposed ticket backlog with a ticket ID assigned to every ticket
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
disable-model-invocation: true
---

# codely.backlog

> **Scope guardrail:** never writes or modifies application code, and never **writes or modifies** any file outside `.sdlc/backlog.md` and `.claude/includes/ticket-format.md`. It reads the SDLC design docs (`.sdlc/*.md`), `CLAUDE.md`, and `.claude/templates/backlog.md`, but treats all of them as read-only. This command makes no git changes and never creates tickets on any real dev board — its output is a proposal, with proposed ticket IDs, for a human to turn into real tickets on whatever tracker this project uses (Jira/Linear/GitHub Issues/etc.).

Turn the approved SDLC design into a proposed ticket backlog at `.sdlc/backlog.md` — one entry per proposed ticket, each assigned a ticket ID that follows this project's `.claude/includes/ticket-format.md` scheme, each traceable back to the SRS's functional requirements and the LLD's components/endpoints, grouped into dependency-ordered waves. This is the bridge between the SDLC design phases and the ticket-by-ticket `codely.*` implementation loop: once a ticket from this backlog exists on your board, run `/codely.spec <ticketId> <description>` to start it. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Phase 0 — Load the design

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md`. **Missing either:** tell the user backlog generation needs an approved system design first, stop, recommend `/sdlc.system-design`.
3. Read both, plus `.sdlc/srs.md` and `.sdlc/use-case-model.md` — functional/non-functional requirements and use cases/actors give the "why" behind each proposed ticket; components and API endpoints give the "what."
4. Glob `.sdlc/project-charter.md` and `.sdlc/project-plan.md` — present → read them too (the charter carries the project name/pitch used to derive the ticket prefix in Phase 1; the plan's roadmap phasing can inform how tickets group into waves).
5. `$ARGUMENTS` non-empty: treat it as extra focus for this pass (e.g. "just the auth module" to extend an existing backlog incrementally).

## Phase 1 — Load the ticket ID format and fix the project key

1. Read `.claude/includes/ticket-format.md`.
2. **Missing** (first run): tell the user the backlog needs the project's ticket ID scheme, then AskUserQuestion — "describe the format, I'll draft the file" vs. "I'll set it up myself, stop here." If they describe it, ask the scheme (JIRA/GitHub/Linear/none/other) and write `.claude/includes/ticket-format.md`. This command makes no git changes — tell the user to commit that file themselves. If they'd rather set it up manually, stop.
3. **Present:** extract the ID shape — whether an ID is `<project-key><separator><number>` (JIRA/Linear-style), a bare number (GitHub Issues), or a placeholder scheme (`NOTICKET-<date>` / none) — plus the project-key rule (allowed characters, length).
4. Fix the **project key** to use for every ticket in this backlog:
   - The project key is **already established** — `CLAUDE.md` names one, an existing `.sdlc/backlog.md` uses one, or `.specs/<key>-*` / `.specs/<key>_*` directories exist; or `.claude/includes/ticket-format.md` has been customized for this project (its "replace the example" setup comment removed) and its examples all share one key → reuse that exact key, unchanged. A key that appears **only** in the shipped template's example (setup comment still present) is not established — derive one instead. Never introduce a second key for the same project.
   - Otherwise derive a 2-3 letter uppercase prefix from the project name (from `.sdlc/project-charter.md`'s title, else `CLAUDE.md`, else the repository folder name): initials for a multi-word name ("Exam Master" → `EXM`), or the first 2-3 letters for a single-word name ("Examester" → `EXA`). It must satisfy the format's project-key rule. AskUserQuestion to confirm — offer the derived prefix and a "let me choose a different one" option.
   - Bare-number format → there is no prefix; IDs are plain sequential integers.
5. Fix the **starting number**: `1`, unless numbers are already in use — for a keyed scheme, this key already appears in an existing `.sdlc/backlog.md` or in `.specs/<key>-*` / `.specs/<key>_*` directory names; for a bare-number scheme, any numeric ticket ID appears in an existing `.sdlc/backlog.md` or as a `.specs/<n>_*` directory. Then start at the highest number already used, plus 1, so IDs never collide with work already begun. (A date-placeholder or no-ticket scheme has no running number — skip this step.)

## Phase 2 — Check for an existing backlog

1. Glob `.sdlc/backlog.md`.
2. **Missing:** proceed to Phase 3.
3. **Found:** read and summarize it — existing tickets, their IDs, which waves. AskUserQuestion — Refresh (add tickets for anything new in the design since this was last generated; keep existing entries and their IDs untouched, and give new tickets the next free numbers per Phase 1) / Start over (regenerate the whole backlog and renumber from Phase 1's starting number) / Cancel.
4. **Legacy shape** — if the found backlog predates the current template (a `Board ID` column instead of `Ticket ID`, and/or IDs embedded in titles like `(EX-1) …`): on **Refresh**, first normalize it to `.claude/templates/backlog.md`'s shape — move each existing ID that already matches `.claude/includes/ticket-format.md` into the `Ticket ID` column and strip it from the title, keeping that ID unchanged; for any row whose ID is missing or doesn't match the format, assign an ID per Phase 5 step 1 (the next free number for a keyed/bare-number scheme; the placeholder representation for a date-placeholder/none scheme — no numeric allocation). On **Start over**, ignore the old IDs entirely and renumber from scratch.

## Phase 3 — Derive candidate tickets

1. Cross-reference the SRS's Functional Requirements with the LLD's Component Design and API Endpoints — group related requirements by the component(s) that implement them.
2. Fold in the HLD's UI/UX Overview (if present) so UI-only work isn't missed.
3. For each candidate ticket, work out: a title, a one-line description, which FR(s)/component(s)/use case(s) it covers, a rough size (S/M/L, based on files/components touched), and what other candidate tickets it depends on.
4. Group candidates into dependency-ordered waves (a ticket goes in the first wave where all its dependencies are satisfied) — align with the project plan's roadmap phases where one exists, but dependencies win over any mismatch.
5. Don't assign ticket IDs yet — the set and its order can still change in Phase 4. Refer to candidates by title until then.

## Phase 4 — Review with the user (rounds)

**Mandatory — don't write the backlog until the user has confirmed the slicing, sizing, and sequencing.**

1. Present the candidate tickets grouped by wave. AskUserQuestion — confirm as-is, or adjust (split, merge, reprioritize, add missing, drop out-of-scope work) — cap 4 options/call, multiple rounds if needed.
2. Digest each round; iterate until the user is satisfied with the slicing.
3. On "you decide" calls (sizing ties, sequencing calls), state a recommendation and get explicit confirmation — record it under Decisions.

## Phase 5 — Assign IDs and write the backlog

1. The ticket set and wave order are now final. Assign every ticket its ID per Phase 1's scheme:
   - **Keyed or bare-number scheme:** walk the tickets in wave order (Wave 1 first), and within a wave in table order, giving each the next number — `<projectKey><separator><n>` (keyed) or a bare `<n>` (number-only) — starting at Phase 1's starting number. IDs are contiguous, one per ticket, never reused. On a Refresh (Phase 2), leave existing tickets' IDs alone and number only the new ones, continuing from the highest number already used.
   - **Date-placeholder scheme (`NOTICKET-<date>` / none):** no numeric allocation. Give every ticket the format's placeholder representation — `NOTICKET-<yyyymmdd>` for today's date, or the "no ticket" form `ticket-format.md` defines. These are non-unique stand-ins the user replaces with real keys when the tickets are cut; say so in the backlog's "How to use this" note.
2. Render `.claude/templates/backlog.md` into `.sdlc/backlog.md`, filling its placeholders — the template file itself stays untouched.
3. Mandatory: at least one Wave table — repeat `## Wave N: {{title}}` with its own table for each additional wave. Each row carries: Ticket ID (from step 1), Title, Description, Source (FR/component/use-case refs), Depends On (the ticket IDs this one depends on, or blank), Size. Tickets within a wave have no dependencies on each other; later waves depend on earlier ones.
4. Optional (omit a section entirely when empty): Out of Scope / Deferred (candidate work identified but deliberately excluded from this backlog — e.g. explicitly out of scope per the charter, or deferred to a later phase of the project plan), Decisions ("you decide" calls made during the review, with rationale — sizing ties, sequencing calls, slicing choices).
5. Write to `.sdlc/backlog.md`.

## Phase 6 — Approval loop & handoff

1. Ask the user to review `.sdlc/backlog.md` — approve or request changes, loop until approved. A requested change that adds, removes, splits, merges, or reorders tickets → go back through Phase 5 so the IDs are reassigned contiguously in wave order (on a Refresh, existing tickets keep their IDs and only the new ones are numbered).
2. No git action follows approval — this command never branches, commits, or pushes, and never touches any external tracker.
3. Confirm the artifact path. Tell the user: create each ticket on your team's actual dev board using the Ticket ID shown (set it as the issue key where the tracker allows, otherwise record it in the tracker's ID/reference field), then run `/clear` then `/codely.spec <ticketId> <description>` per ticket, in the wave order shown, to start implementation.
