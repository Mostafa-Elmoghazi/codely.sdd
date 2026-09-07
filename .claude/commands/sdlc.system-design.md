---
description: Turn an approved SRS/use-case model into an approved HLD + LLD
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
disable-model-invocation: true
---

# sdlc.system-design

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/hld.md` and `.sdlc/lld.md`. This command makes no git changes (no branch, no commit, no PR) — approval happens in chat.

Turn the approved requirements into two approved artifacts under `.sdlc/`:

- **`hld.md`** — High-Level Design: architecture, major modules, data model, technology selection, and a light-touch UI/UX overview.
- **`lld.md`** — Low-Level Design: component/class design and API endpoint contracts.

This is the last SDLC-phase command — its output is the blueprint that `codely.*` commands (starting with `/codely.spec`) turn into tickets and code. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

## Phase 0 — Load the requirements

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/srs.md`. **Missing:** tell the user System Design needs an approved SRS first, stop, recommend `/sdlc.req-analysis`.
3. Glob `.sdlc/use-case-model.md`. **Missing:** same stop/recommend.
4. Read both.
5. Glob `.sdlc/project-charter.md` — present → read it too (its Feasibility/constraints notes can already bound technology choices).
6. `$ARGUMENTS` non-empty: treat it as extra focus/context for this pass (e.g. narrowing to a subsystem, or constraints from a since-held infra discussion).

## Phase 1 — Check for existing artifacts

1. Glob `.sdlc/hld.md` and `.sdlc/lld.md`.
2. **Both missing:** proceed to Phase 2.
3. **Either found:** read whichever exist(s), including their `## Status` checklists.
   - **Every found topic checked:** summarize both, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate both) / Cancel.
   - **Some topics unchecked (a paused mid-interview session):** summarize what's already settled and list what's still open, AskUserQuestion — Continue (resume Phase 3 at the first unchecked topic, keeping everything already settled) / Start over (discard both drafts, redo from scratch) / Cancel.

## Phase 2 — Context gathering

Glob/Grep/Read the repo for existing signal before assuming greenfield: package manifests, IaC/config files, existing folder structure, ADRs, or prior ticket-level `.specs/<ticketId>_*/system-design.md` and `design.md` files. Design around what's already real; don't ask the user what the repo already answers.

## Phase 3 — Interview & draft (topic by topic)

**Mandatory — don't move to Phase 4 until every topic below is checked off.**

Work through the topics below **one at a time, in the order listed** — each topic's answer is written to disk as soon as it's resolved, so the work is never lost mid-interview and a later session can pick up exactly where this one stopped.

For each topic:

1. Ask about it via AskUserQuestion round(s) (cap 4/call — split a 5-question topic across two calls). Digest answers; ask a follow-up round if new questions surface. On "you decide," state a recommendation and get explicit confirmation — record it under that artifact's Decisions.
2. If the topic turns out to be multi-part or is taking several rounds, split its Status entry into indented sub-items and check those off individually, checking the parent once all its sub-items (or the topic as a whole) are settled — resolve them one at a time.
3. The first time a topic belonging to a given artifact is resolved, create that artifact from its template (`.claude/templates/hld.md` / `.claude/templates/lld.md`) with its `## Status` checklist intact — the checklist tracks interview progress across sessions so work can pause and resume without redoing settled topics; safe to ignore once every box is checked, and never delete the section. Write/update that topic's section, then check it off (and its parent, if all sub-items are done).
4. A later topic changes an earlier decision → update that earlier section too; its Status box stays checked (the box tracks "resolved at least once," not "immutable").
5. Move to the next topic.

**`.sdlc/hld.md` topics** (in order):

- **Architecture Overview** — overall style (monolith/microservices/serverless), major modules/services and their responsibilities, how they communicate; also covers Major Modules (one entry per module/service). In the flowchart diagram, keep node IDs plain and quote every label holding a placeholder (Mermaid treats a bare `{` as a diamond-node token even inside brackets).
- **Data Model** — key entities, relationships, storage choice(s), as an entity-relationship diagram. Entity names and attribute types/names must stay plain identifiers (Mermaid's ER grammar doesn't allow braces there) — only the relationship label may hold a placeholder, since it's a quoted string.
- **Technology Selection** — languages, frameworks, hosting/infra — tied to the SRS's non-functional requirements (performance, scale, compliance, etc.).
- **UI/UX Overview** — key screens and high-level navigation only, if the project has a user-facing UI; detailed mockups are deferred to `/codely.ui` per ticket. Skip (check off with a one-line note) if there's no user-facing UI.

**`.sdlc/lld.md` topics** (in order, after the HLD's topics):

- **Component Design** — one entry per module from the HLD, detailed enough for developers to implement directly, each with its own class diagram — repeat `### {{component_name}}` with its own Responsibility/diagram for each additional component. Class/attribute/method names stay plain identifiers (Mermaid's class-diagram grammar doesn't allow braces there).
- **API Endpoints** — style (REST/GraphQL/RPC) and key endpoints. Skip (check off with a one-line note) if the system exposes no API.
- **Cross-cutting Concerns** — auth, error handling, observability, where relevant; lands mainly in the (optional — omit if none beyond standard conventions) Error Handling section and the API table's Auth column, and may also touch the HLD's Traceability.

Once all HLD topics are checked, (re)synthesize the HLD's Traceability table (mapping decisions back to the SRS's requirements and the use-case model's actors/workflows) — it isn't interviewed as its own topic.

## Phase 4 — Approval loop

1. Confirm every Status item across both artifacts is checked — any left open, return to Phase 3 for just those topics.
2. Ask the user to review both `.sdlc/hld.md` and `.sdlc/lld.md` — approve or request changes, loop until approved.
3. No git action follows approval — this command never branches, commits, or pushes.
4. Confirm both artifact paths. Tell the user the SDLC phases are complete — run `/clear` then, in either order, `/codely.sys-init` (turns this design into a real, phased project scaffold) and/or `/codely.backlog` (turns it into a proposed ticket backlog), then `/codely.spec` per ticket to start implementation.
