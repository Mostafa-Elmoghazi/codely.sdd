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

## Phase 0 — Load the charter & plan

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/project-charter.md`. **Missing:** tell the user Requirements Analysis needs an approved charter first, stop, recommend `/sdlc.plan`.
3. Read `.sdlc/project-charter.md`.
4. Glob `.sdlc/project-plan.md` — present → read it too (the roadmap's phasing can inform requirement priority and sequencing).
5. `$ARGUMENTS` non-empty: treat it as extra focus/context for this pass (e.g. narrowing to a subsystem, or notes from a stakeholder meeting).

## Phase 1 — Check for existing artifacts

1. Glob `.sdlc/srs.md` and `.sdlc/use-case-model.md`.
2. **Both missing:** proceed to Phase 2.
3. **Either found:** read and summarize them, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate both) / Cancel.

## Phase 2 — Context gathering

Glob/Grep/Read the repo for existing signal: `README.md`, existing `docs/`, and any prior ticket-level specs under `.specs/` (these can hint at actors, use cases, or conventions already implemented). Don't ask the user what the repo already answers.

## Phase 3 — Interview the user (rounds)

**Mandatory — don't draft either artifact until actors, use cases, functional/non-functional requirements, and workflows are all clear.**

1. List open questions from the charter/plan + Phase 2 findings, across the core requirements-analysis activities:
   - **Actors** — who or what interacts with the system (end users, admins, external systems).
   - **Use Cases** — the key interactions each actor has with the system, and their goals.
   - **User Stories** — concrete "as a ⟨role⟩, I want ⟨capability⟩, so that ⟨benefit⟩" statements.
   - **Process Workflows** — step-by-step flows for key or cross-actor processes.
   - **Functional Requirements** — specific, testable behaviors.
   - **Non-Functional Requirements** — performance, security, compliance, accessibility, scalability, etc.
   - **Validation** — confirm the above aligns with the charter's goals and scope.
2. AskUserQuestion in rounds of up to 4-5 (cap is 4/call — split a 5-question round across two calls). Clear labels, short trade-offs.
3. Digest each round; run another if new questions surface.
4. On "you decide," state a recommendation and get explicit confirmation — record it under Decisions.

## Phase 4 — Write the use-case model

1. Fill `.claude/templates/use-case-model.md` placeholders.
2. Mandatory: Actors, Use Cases (with the Mermaid use-case/system-boundary diagram), Process Workflows (with Mermaid sequence diagrams).
3. Optional — omit the whole section if empty, never write "N/A": User Stories, Notes & Decisions.
4. Write to `.sdlc/use-case-model.md`.

## Phase 5 — Write the SRS

1. Fill `.claude/templates/srs.md` placeholders.
2. Mandatory: Purpose & Scope, Functional Requirements, Non-Functional Requirements, Traceability (map requirements back to the charter's goals).
3. Optional — omit the whole section if empty, never write "N/A": Assumptions & Constraints, Decisions.
4. Write to `.sdlc/srs.md`.

## Phase 6 — Approval loop

1. Ask the user to review both `.sdlc/use-case-model.md` and `.sdlc/srs.md` — approve or request changes, loop until approved.
2. No git action follows approval — this command never branches, commits, or pushes.
3. Confirm both artifact paths. Tell the user to run `/clear` then `/sdlc.system-design` next to turn these into a system design.
