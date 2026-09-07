---
description: Turn a project idea into an approved project charter + project plan (goals, scope, feasibility, roadmap, resourcing, risk)
argument-hint: [brief project pitch]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion
disable-model-invocation: true
---

# sdlc.plan

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/project-charter.md` and `.sdlc/project-plan.md`. This command makes no git changes (no branch, no commit, no PR) — approval happens in chat.

Turn a project idea into two approved artifacts under `.sdlc/`:

- **`project-charter.md`** — why this project exists: goals, scope, feasibility, risk.
- **`project-plan.md`** — how it gets delivered: roadmap, milestones, resourcing.

Together these are the foundation the Requirements Analysis phase (`/sdlc.req-analysis`) builds on next. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

## Phase 0 — Resolve the pitch

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: treat it as the initial project pitch (name + brief description, free-form).
3. `$ARGUMENTS` empty: ask the user for a one- or two-line pitch — what's being built and why. Don't continue without it.

## Phase 1 — Check for existing artifacts

1. Glob `.sdlc/project-charter.md` and `.sdlc/project-plan.md`.
2. **Both missing:** proceed to Phase 2.
3. **Either found:** read whichever exist(s), including their `## Status` checklists.
   - **Every found topic checked:** summarize both, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate both) / Cancel.
   - **Some topics unchecked (a paused mid-interview session):** summarize what's already settled and list what's still open, AskUserQuestion — Continue (resume Phase 3 at the first unchecked topic, keeping everything already settled) / Start over (discard both drafts, redo from scratch) / Cancel.

## Phase 2 — Context gathering

Glob/Grep/Read the repo root for existing signal: `README.md`, dependency manifests, existing `docs/`, prior planning notes. Don't ask the user what the repo already answers (e.g. an obvious tech stack, an existing product description).

## Phase 3 — Interview & draft (topic by topic)

**Mandatory — don't move to Phase 4 until every topic below is checked off.**

Work through the topics below **one at a time, in the order listed** — each topic's answer is written to disk as soon as it's resolved, so the work is never lost mid-interview and a later session can pick up exactly where this one stopped.

For each topic:

1. Ask about it via AskUserQuestion round(s) (cap 4/call — split a 5-question topic across two calls). Digest answers; ask a follow-up round if new questions surface. On "you decide," state a recommendation and get explicit confirmation — record it under that artifact's Decisions.
2. If the topic turns out to be multi-part or is taking several rounds, split its Status entry into indented sub-items and check those off individually, checking the parent once all its sub-items (or the topic as a whole) are settled — resolve them one at a time.
3. The first time a topic belonging to a given artifact is resolved, create that artifact from its template (`.claude/templates/project-charter.md` / `.claude/templates/project-plan.md`, creating `.sdlc/` if needed) with its `## Status` checklist intact — the checklist tracks interview progress across sessions so work can pause and resume without redoing settled topics; safe to ignore once every box is checked, and never delete the section. Write/update that topic's section, then check it off (and its parent, if all sub-items are done).
4. A later topic changes an earlier decision → update that earlier section too; its Status box stays checked (the box tracks "resolved at least once," not "immutable").
5. Move to the next topic.

**`.sdlc/project-charter.md` topics** (in order):

- **Goals** — the core business problem or opportunity, and what success looks like.
- **Scope** — project boundaries and high-level deliverables.
- **Feasibility Assessment** — technical, economic, legal, and operational viability.
- **Risks & Mitigations** — potential roadblocks and how they'll be managed.

**`.sdlc/project-plan.md` topics** (in order, after the charter's topics):

- **Roadmap** — phases run in order, each a milestone-bearing chunk of work; repeat `### Phase N: {{title}}` with its own Timeline/Milestone/Deliverables for each additional phase (a single-phase roadmap still uses the `Phase 1: <title>` heading). Also covers the Strategy section (high-level approach to delivering the charter's goals — how the roadmap was sequenced and why) and the Milestone Checklist rollup (checklist of milestones/checkpoints that mark this plan's completion — the gate before `/sdlc.req-analysis` begins).
- **Resource Allocation** — budget, team composition/skills, and tooling required.

Optional sections aren't tracked topics — fill them in opportunistically whenever relevant information surfaces, omit the whole section if empty, never write "N/A": Out of Scope (what the charter explicitly doesn't cover), Stakeholders, Assumptions & Constraints for the charter; Dependencies (cross-phase or external dependencies — vendors, approvals, hires — that could block sequencing) for the plan.

## Phase 4 — Approval loop

1. Confirm every Status item across both artifacts is checked — any left open, return to Phase 3 for just those topics.
2. Ask the user to review both `.sdlc/project-charter.md` and `.sdlc/project-plan.md` — approve or request changes, loop until approved.
3. No git action follows approval — this command never branches, commits, or pushes.
4. Confirm both artifact paths. Tell the user to run `/clear` then `/sdlc.req-analysis` next to turn these into a Software Requirements Specification.
