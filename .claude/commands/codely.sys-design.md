---
description: Turn an approved spec into a ticket-level system design (architecture/data model/API contract/components)
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(git:*)
disable-model-invocation: true
---

# codely.sys-design

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.specs/<ticketId>_<slug>/system-design.md`. Outputs are the system-design file and — after approval — a commit (no push) on the ticket's existing docs branch.

**Optional step.** Turn an approved spec into a ticket-level system design at `.specs/<ticketId>_<slug>/system-design.md`, then commit it (no push) to the ticket's docs branch. Skip this command for tickets that don't need explicit architecture/data/API design beyond the spec — `/codely.plan`, `/codely.implement`, `/codely.qa`, and `/codely.ui` already treat this file as "present → read it," so nothing downstream breaks if it's absent. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`

## Phase 0 — Resolve the ticket

1. Read `CLAUDE.md` if present.
2. Current branch matches `<ticketId>/<slug>/feature` (validate `ticketId` against `.claude/includes/ticket-format.md` if present) → extract `ticketId` and `slug` — this is the docs branch.
3. Trunk, or non-matching branch → use `$ARGUMENTS`' first token if given, else — inside a ticket worktree (see `.claude/includes/worktree-ticket.md`) — the worktree's ticket ID, else ask. Don't guess.
4. Locate `.specs/<ticketId>_*`. Missing → tell the user, stop, recommend `/codely.spec`. Found → the directory's slug is the source of truth from here on.
5. `.specs/<ticketId>_<slug>/spec.md` missing → stop, tell the user `/codely.sys-design` needs an approved spec first, recommend `/codely.spec`.
6. Not already on `<ticketId>/<slug>/feature` → dirty working tree → stop, AskUserQuestion (commit / stash / discard / abort); otherwise switch to it.

## Phase 1 — Load context

1. Read `spec.md` — functional/non-functional requirements, constraints, and acceptance criteria bound the design.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md` — present → read them too, so this ticket's architecture, data model, and API/component choices stay consistent with the project-wide design rather than drifting from it.
3. `system-design.md` already exists → summarize it, AskUserQuestion — Resume/revise (edit in place, skip re-drafting from scratch) / Start over (regenerate) / Cancel.

## Phase 2 — Codebase exploration

@.claude/includes/claude-md-read-first.md

Glob/Grep/Read the existing codebase — current architecture, existing data models/schemas, existing API routes, existing component/class patterns. This ticket's design should extend what's real, not invent a parallel structure. Don't ask the user what the code already answers.

## Phase 3 — Interview the user (rounds)

**Mandatory — don't draft the system design until the architecture approach, data model changes, API contract, and component breakdown are all clear.**

Start from the open questions raised by spec gaps + `.sdlc/` findings + Phase 2 exploration, across:

- **Architecture** — how this ticket's changes fit the existing (or project HLD) architecture; any new components/services.
- **Data Model** — new or changed entities, fields, relationships, migrations.
- **API Contract** — new or changed endpoints, request/response shapes, auth.
- **Component Breakdown** — key modules/classes this ticket touches or introduces.

Then:

@.claude/includes/interview.md

Record any "you decide" call under Decisions.

## Phase 4 — Write the system design

1. Fill `.claude/templates/system-design.md` placeholders.
2. Mandatory: Architecture (how this ticket's changes fit the existing system, or the project's `.sdlc/hld.md` if it exists — new or changed components and how they communicate), Component Breakdown (key modules/classes this ticket touches or introduces).
3. Mandatory — omit only if this ticket introduces no such changes: Data Model (new or changed entities, fields, relationships), API Contract.
4. Optional (omit a section entirely when empty): Decisions ("you decide" calls made during the interview, with rationale).
5. In every Mermaid diagram, keep node/entity/class/attribute/method names plain identifiers and quote every label holding a placeholder (Mermaid treats a bare `{` as a diamond-node token even inside brackets, and its ER/class grammar doesn't allow braces in identifiers) — only an edge or relationship label may hold a placeholder, since it's already a quoted string.
6. Write to `.specs/<ticketId>_<slug>/system-design.md`.

## Phase 5 — Approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/system-design.md` — approve or request changes, loop until approved.
2. Approval is a chat-only gate for Phase 6 — never add an approver name to the file.

## Phase 6 — Commit

**Don't start until Phase 5 approval is explicit.**

1. `git status --porcelain`, disregarding `system-design.md` just written — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort).
2. Stage `system-design.md` only.
3. Commit: `Add system design for <ticketId>`.
4. Don't push — `/codely.plan`'s existing push step will carry this commit along with `design.md`/`plan.md` into one PR reviewing spec + design + plan together.
5. Confirm: `system-design.md` path. Tell the user to run `/clear` then `/codely.ui` (optional, for UI-heavy tickets) or `/codely.plan` next.
