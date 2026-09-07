---
description: Turn an approved project charter/plan into an approved SRS + use-case model
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
disable-model-invocation: true
---

# sdlc.req-analysis

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/srs.md` and `.sdlc/use-case-model.md`. This command makes no git changes (no branch, no commit, no PR) — approval happens in chat.

Turn the approved project charter into two approved artifacts under `.sdlc/`:

- **`use-case-model.md`** — actors, use cases, user stories, and process workflows (with Mermaid diagrams).
- **`srs.md`** — functional & non-functional requirements, traceable back to the charter's goals.

Together these are the blueprint the System Design phase (`/sdlc.system-design`) builds on next. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

## Phase 0 — Load the charter & plan

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/project-charter.md`. **Missing:** tell the user Requirements Analysis needs an approved charter first, stop, recommend `/sdlc.plan`.
3. Read `.sdlc/project-charter.md`.
4. Glob `.sdlc/project-plan.md` — present → read it too (the roadmap's phasing can inform requirement priority and sequencing).
5. `$ARGUMENTS` non-empty: treat it as extra focus/context for this pass (e.g. narrowing to a subsystem, or notes from a stakeholder meeting).

## Phase 1 — Check for existing artifacts

1. Glob `.sdlc/srs.md` and `.sdlc/use-case-model.md`.
2. **Both missing:** proceed to Phase 2.
3. **Either found:** read whichever exist(s), including their `## Status` checklists.
   - **Every found topic checked:** summarize both, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate both) / Cancel.
   - **Some topics unchecked (a paused mid-interview session):** summarize what's already settled and list what's still open, AskUserQuestion — Continue (resume Phase 3 at the first unchecked topic, keeping everything already settled) / Start over (discard both drafts, redo from scratch) / Cancel.

## Phase 2 — Context gathering

Glob/Grep/Read the repo for existing signal: `README.md`, existing `docs/`, and any prior ticket-level specs under `.specs/` (these can hint at actors, use cases, or conventions already implemented). Don't ask the user what the repo already answers.

## Phase 3 — Interview & draft (topic by topic)

**Mandatory — don't move to Phase 4 until every topic below is checked off.**

Work through the topics below **one at a time, in the order listed** — each topic's answer is written to disk as soon as it's resolved, so the work is never lost mid-interview and a later session can pick up exactly where this one stopped.

For each topic:

1. Ask about it via AskUserQuestion round(s) (cap 4/call — split a 5-question topic across two calls). Digest answers; ask a follow-up round if new questions surface. On "you decide," state a recommendation and get explicit confirmation — record it under that artifact's Decisions (or Notes & Decisions for the use-case model).
2. If the topic turns out to be multi-part or is taking several rounds, split its Status entry into indented sub-items and check those off individually, checking the parent once all its sub-items (or the topic as a whole) are settled — resolve them one at a time.
3. The first time a topic belonging to a given artifact is resolved, create that artifact from its template (`.claude/templates/use-case-model.md` / `.claude/templates/srs.md`) with its `## Status` checklist intact — the checklist tracks interview progress across sessions so work can pause and resume without redoing settled topics; safe to ignore once every box is checked, and never delete the section. Write/update that topic's section, then check it off (and its parent, if all sub-items are done).
4. A later topic changes an earlier decision → update that earlier section too; its Status box stays checked (the box tracks "resolved at least once," not "immutable").
5. Move to the next topic.

**`.sdlc/use-case-model.md` topics** (in order):

- **Actors** — who or what interacts with the system (end users, admins, external systems).
- **Use Cases** — key interactions between actors and the system, with the Mermaid use-case/system-boundary diagram: Mermaid has no native UML use-case shape, so actors and use cases are approximated as a flowchart — use cases as stadium nodes inside a system-boundary subgraph, actors as plain nodes connected to the use cases they perform; repeat/extend the diagram, or add further diagrams per subsystem, as use cases grow, and repeat `### {{use_case_title}}` with its own Actor(s)/Goal/Preconditions/Flows for each additional use case.
- **User Stories** — optional content (omit the section if empty, `"As a ⟨role⟩, I want ⟨capability⟩, so that ⟨benefit⟩."`) captured separately from the SRS's Functional Requirements, but stays a tracked topic — check it off once considered, whether or not it produced content separate from the use cases.
- **Process Workflows** — step-by-step flows for key or cross-actor processes, as Mermaid sequence diagrams; repeat `### {{workflow_title}}` with its own sequence diagram for each additional workflow.

**`.sdlc/srs.md` topics** (in order, after the use-case model's topics):

- **Functional Requirements** — specific, testable behaviors the system must implement, grouped by feature area or use case; repeat `### {{requirement_group}}` for each additional group.
- **Non-Functional Requirements** — performance, security, compliance, accessibility, scalability, etc.
- **Validation** — confirm the drafted use cases and requirements align with the charter's goals and scope; use this pass to also (re)synthesize the SRS's Purpose & Scope (what this SRS covers, and how it traces back to the charter's goals and scope) and Traceability (map requirements back to the charter's goals, so nothing drifts from why the project exists) sections from everything gathered so far, since those aren't interviewed as their own topics.

Assumptions & Constraints (srs) is optional and not a tracked topic — fill it in opportunistically whenever relevant information surfaces, omit if empty, never write "N/A".

## Phase 4 — Approval loop

1. Confirm every Status item across both artifacts is checked — any left open, return to Phase 3 for just those topics.
2. Ask the user to review both `.sdlc/use-case-model.md` and `.sdlc/srs.md` — approve or request changes, loop until approved.
3. No git action follows approval — this command never branches, commits, or pushes.
4. Confirm both artifact paths. Tell the user to run `/clear` then `/sdlc.system-design` next to turn these into a system design.
