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

## Phase 0 — Resolve the pitch

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: treat it as the initial project pitch (name + brief description, free-form).
3. `$ARGUMENTS` empty: ask the user for a one- or two-line pitch — what's being built and why. Don't continue without it.

## Phase 1 — Check for existing artifacts

1. Glob `.sdlc/project-charter.md` and `.sdlc/project-plan.md`.
2. **Both missing:** proceed to Phase 2.
3. **Either found:** read and summarize them, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate both) / Cancel.

## Phase 2 — Context gathering

Glob/Grep/Read the repo root for existing signal: `README.md`, dependency manifests, existing `docs/`, prior planning notes. Don't ask the user what the repo already answers (e.g. an obvious tech stack, an existing product description).

## Phase 3 — Interview the user (rounds)

**Mandatory — don't draft either artifact until goals, scope, feasibility, roadmap, resourcing, and risk are all clear.**

1. List open questions from the Phase 0 pitch + Phase 2 findings, across the core planning activities:
   - **Goals** — the core business problem or opportunity, and what success looks like.
   - **Scope** — project boundaries and high-level deliverables (what's in, what's explicitly out).
   - **Feasibility** — technical, economic, legal, and operational viability.
   - **Roadmap** — how the work breaks into phases/milestones, and their rough sequencing and timeline.
   - **Resources** — budget, team composition/skills, and tooling required.
   - **Risks** — potential roadblocks and mitigation strategies.
2. AskUserQuestion in rounds of up to 4-5 (cap is 4/call — split a 5-question round across two calls). Clear labels, short trade-offs.
3. Digest each round; run another if new questions surface.
4. On "you decide," state a recommendation and get explicit confirmation — record it under Decisions.

## Phase 4 — Write the charter

1. Fill `.claude/templates/project-charter.md` placeholders.
2. Mandatory: Goals, Scope, Feasibility Assessment, Risks & Mitigations.
3. Optional — omit the whole section if empty, never write "N/A": Out of Scope, Stakeholders, Assumptions & Constraints, Decisions.
4. Write to `.sdlc/project-charter.md`, creating `.sdlc/` if needed.

## Phase 5 — Write the plan

1. Fill `.claude/templates/project-plan.md` placeholders.
2. Mandatory: Strategy, Roadmap (each phase with Timeline/Milestone/Deliverables), Resource Allocation, Milestone Checklist.
3. Optional — omit the whole section if empty, never write "N/A": Dependencies.
4. A single-phase roadmap still uses the `Phase 1: <title>` heading.
5. Write to `.sdlc/project-plan.md`.

## Phase 6 — Approval loop

1. Ask the user to review both `.sdlc/project-charter.md` and `.sdlc/project-plan.md` — approve or request changes, loop until approved.
2. No git action follows approval — this command never branches, commits, or pushes.
3. Confirm both artifact paths. Tell the user to run `/clear` then `/sdlc.req-analysis` next to turn these into a Software Requirements Specification.
