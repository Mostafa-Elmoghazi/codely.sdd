---
description: Turn an approved spec into previewable screen mockups and a design breakdown
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash(git:*), Artifact, EnterWorktree, ExitWorktree
disable-model-invocation: true
---

# codely.ui

> **Scope guardrail:** never writes or modifies application code — outputs are static, self-contained HTML/CSS mockups, not framework components. Within `.specs/<ticketId>_<slug>/`, touches only `design.md` and the `design/` folder — never `spec.md`. Published Artifact preview links are a live-iteration convenience for this session, not the durable output — the committed `design/*.html` files are. Outputs are the design files and — after approval — a commit on the ticket's existing docs branch (no push; `/codely.plan`'s push step carries it into the spec+design+plan PR).

Turn an approved spec into a set of previewable screen mockups plus a design breakdown at `.specs/<ticketId>_<slug>/design.md`, then commit them (no push) to the ticket's docs branch. Follow the phases in order. No git changes before approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

@.claude/includes/no-ai-attribution.md

@.claude/includes/git-message-format.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Worktrees: !`git worktree list 2>/dev/null || echo "(none)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`

## Phase 0 — Resolve the ticket

@.claude/includes/worktree-ticket.md

1. Read `CLAUDE.md` if present.
2. Determine `ticketId`, first match wins (validate against `.claude/includes/ticket-format.md` if present): `$ARGUMENTS`' first token → inside a ticket worktree, the worktree's ticket ID → the current branch, when it's the docs branch `<ticketId>/<slug>/feature` → otherwise ask. Don't guess.
3. Locate `.specs/<ticketId>_*`. Missing → tell the user, stop, recommend `/codely.spec`. Found → the directory's slug is the source of truth from here on.
4. `.specs/<ticketId>_<slug>/spec.md` missing → stop, tell the user `/codely.ui` needs an approved spec first, recommend `/codely.spec`.
5. Not already on `<ticketId>/<slug>/feature` → dirty working tree → stop, AskUserQuestion (commit / stash / discard / abort); otherwise switch to it.

## Phase 1 — Load context

1. Read `spec.md` — functional requirements, user stories, and acceptance criteria are the brief for what screens are needed.
2. `system-design.md` present → read it too (data shapes / API constraints that bound what the UI can show).
3. `design.md` already exists → summarize its screens, AskUserQuestion — Resume/revise (add or revise screens, keep the rest) / Start over (regenerate all) / Cancel.

## Phase 2 — Design system discovery

@.claude/includes/claude-md-read-first.md

1. Glob `.sdlc/ui-template/` — present → read `tokens.css` and its screen files, and `.sdlc/ui-template.md`'s Theme Direction/Design Tokens sections; this is the project's established design system, so mockups must reuse it. Still check the live codebase (next step) too, in case real conventions have since diverged from it.
2. Glob/Grep/Read the codebase for existing design conventions — CSS variables/tokens, Tailwind or similar config, theme files, a shared component library, existing screens' markup and styling, brand colors/fonts, any style guide or Storybook. Found (and step 1 found nothing) → mockups must reuse these, not invent a new look. Neither step found anything → say so plainly and use a neutral, unbranded placeholder style, flagged to the user as provisional.

## Phase 3 — Screen inventory

1. From the spec's user stories/requirements/acceptance criteria and Phase 2's findings, propose the set of screens needed to cover the ticket, plus which states matter per screen (empty/loading/error/populated, etc.) and target platform (mobile/website/desktop — per the spec, or ask if unclear).
2. AskUserQuestion — confirm the inventory, or adjust (add/drop screens or states) before drafting.
3. Generate each screen's slug (slug rule per `.claude/includes/slug.md`).

## Phase 4 — Interview the user (rounds)

**Mandatory — don't start drafting mockups until layout intent, key content, primary actions, and navigation between screens are clear.**

Start from the open questions raised by spec gaps + Phase 2/3 findings, then:

@.claude/includes/interview.md

## Phase 5 — Draft and preview each screen

For each screen confirmed in Phase 3:

1. Build a self-contained HTML/CSS mockup: real representative content (not lorem ipsum), the states identified in Phase 3, and Phase 2's design tokens/components — or the flagged neutral placeholder style if none were found.
2. Wrap per platform: mobile → phone-frame viewport; desktop → window-chrome frame; website → plain responsive layout, no wrapper.
3. Publish via the Artifact tool and share the URL.
4. Walk the user through it — layout, content, states, interactions. Revise and republish to the same URL each round until the user confirms this screen. Move to the next screen once confirmed.
5. A screen turns out to need splitting, merging, or dropping mid-draft → update the Phase 3 inventory and tell the user before continuing.

## Phase 6 — Write the design files

1. For each confirmed screen, save its final HTML/CSS to `.specs/<ticketId>_<slug>/design/<screen-slug>.html` (self-contained, matching the last published Artifact revision).
2. Fill `.claude/templates/design.md` placeholders.
3. Mandatory: Screens (one entry per screen — repeat `### {{screen_title}}` with its own File/Platform/States for each additional screen), Component Breakdown (reusable/existing vs. new components identified across the screens).
4. Optional (omit a section entirely when empty): Navigation / Flow (omit if a single screen — how screens connect: entry points, transitions, back/cancel behavior), Non-Functional Notes (responsive breakpoints, accessibility considerations, platform conventions followed), Open Questions (deferred to `follow-up.md` or resolved during `/codely.plan`).
5. Write to `.specs/<ticketId>_<slug>/design.md`.

## Phase 7 — Approval loop

1. Ask the user to review `.specs/<ticketId>_<slug>/design.md` and each `design/<screen-slug>.html` (the last Artifact preview link works too) — approve or request changes, loop until approved. Changes to a specific screen → back to Phase 5 for that screen, then redo Phase 6 for it.
2. Approval is a chat-only gate for Phase 8 — never add an approver name to the file.

## Phase 8 — Commit

**Don't start until Phase 7 approval is explicit.**

1. `git status --porcelain`, disregarding `design.md`/`design/` just written in Phase 6 — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort).
2. Stage `design.md` and the `design/` directory.
3. Commit: `<ticketId>: Add UI design`.
4. Don't push — `/codely.plan`'s existing push step will carry this commit along with `plan.md` into one PR reviewing spec + design + plan together.
5. Confirm: `design.md` path, screen files, and that the published Artifact preview links are a session convenience only (may expire or be revoked) — the committed `design/*.html` files are the durable output. Tell the user to run `/clear` then `/codely.plan` next.
