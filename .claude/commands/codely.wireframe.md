---
description: Turn the approved system design into a low-fidelity, whole-product screen inventory and wireframes
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Artifact
disable-model-invocation: true
---

# codely.wireframe

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/wireframe.md` and `.sdlc/wireframes/`. This command makes no git changes (no branch, no commit, no PR) — approval happens in chat. Published Artifact preview links are a live-iteration convenience for this session, not the durable output — the committed `.sdlc/wireframes/*.html` files are.

**Optional step.** Turn the approved system design into a low-fidelity, whole-product screen inventory, wireframes (structure and layout only — boxes and labels, no visual styling), and navigation flow, at `.sdlc/wireframe.md` + `.sdlc/wireframes/<screen-slug>.html`. Skip this command for projects without a user-facing UI, or where the HLD's UI/UX Overview is already enough to move on from. This establishes the whole-product information architecture that `/codely.backlog` (screens → tickets) and later per-ticket `/codely.ui` runs should stay consistent with, and that `/codely.ui-template` (if run) turns into a high-fidelity design. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Phase 0 — Load context

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md`. **Missing either:** tell the user wireframing needs an approved system design first, stop, recommend `/sdlc.system-design`.
3. Read both — the HLD's UI/UX Overview is the starting point; the LLD's components bound what's feasible per screen.
4. Glob `.sdlc/srs.md` and `.sdlc/use-case-model.md` — present → read them too, functional requirements and use cases/user stories/workflows drive full screen coverage.
5. Glob `.sdlc/project-charter.md` — present → read it too, its Goals/Scope carry the project idea, industry, and target audience used in Phase 4.
6. `$ARGUMENTS` non-empty: treat it as extra focus for this pass (e.g. narrowing to one area of the product).

## Phase 1 — Check for an existing wireframe

1. Glob `.sdlc/wireframe.md`.
2. **Missing:** proceed to Phase 2.
3. **Found:** read and summarize its screens. AskUserQuestion — Resume/revise (add or revise screens, keep the rest) / Start over (regenerate all) / Cancel.

## Phase 2 — Design system discovery

@.claude/includes/claude-md-read-first.md

Glob/Grep/Read the codebase for existing UI conventions — layout patterns, navigation structure, component library, existing screens' markup. Found → wireframes should extend this, not invent a parallel structure (relevant for adding to an existing product). Nothing found → proceed as greenfield.

## Phase 3 — Screen inventory

1. From the HLD's UI/UX Overview, the SRS's functional requirements, and the use-case model's use cases/user stories/workflows, propose the full set of screens needed for the whole product, plus which states matter per screen (empty/loading/error/populated, etc.) and target platform (mobile/website/desktop — per the HLD, or ask if unclear).
2. AskUserQuestion — confirm the inventory, or adjust (add/drop screens or states) before drafting.
3. Generate each screen's slug (slug rule per `.claude/includes/slug.md`).

## Phase 4 — Interview the user (rounds)

**Mandatory — don't start drafting wireframes until the navigation paradigm, information architecture, and primary flows through the product are clear.**

Before asking, synthesize what's already on disk from Phase 0 — the charter's stated idea, industry/domain, and target audience/persona — and _propose_ a layout/IA direction with a one-line rationale (e.g. "a field-service scheduling app used on-site by tradespeople → mobile-first, large tap targets, shallow navigation — does that fit?"). Then work through the remaining open questions from HLD/SRS/use-case-model gaps + Phase 2/3 findings — navigation paradigm (sidebar/top-nav/tab-bar/etc.), information density, primary flows between screens:

@.claude/includes/interview.md

## Phase 5 — Draft and preview each screen

For each screen confirmed in Phase 3:

1. Build a self-contained HTML wireframe: layout blocks and placeholder labels only — grayscale, no color, no typographic polish, no imagery — deliberately low-fidelity, showing structure and the states identified in Phase 3.
2. Wrap per platform: mobile → phone-frame viewport; desktop → window-chrome frame; website → plain responsive layout, no wrapper.
3. Publish via the Artifact tool and share the URL.
4. Walk the user through it — layout, content blocks, states, navigation. Revise and republish to the same URL each round until the user confirms this screen. Move to the next screen once confirmed.
5. A screen turns out to need splitting, merging, or dropping mid-draft → update the Phase 3 inventory and tell the user before continuing.

## Phase 6 — Write the wireframe files

1. For each confirmed screen, save its final HTML to `.sdlc/wireframes/<screen-slug>.html` (self-contained, matching the last published Artifact revision).
2. Fill `.claude/templates/wireframe.md` placeholders.
3. Mandatory: Screens (one entry per screen — repeat `### {{screen_title}}` with its own File/Platform/States for each additional screen), Navigation / Flow (how screens connect: entry points, transitions, back/cancel behavior — always populated here, since this is whole-product and always multi-screen, unlike `/codely.ui`'s per-ticket `design.md` where it's optional).
4. Optional (omit a section entirely when empty): Layout Notes (density/responsive/platform conventions followed), Open Questions (deferred to `/codely.ui-template` or resolved during ticket-level work).
5. In the Navigation / Flow Mermaid diagram, keep node IDs plain and quote every label holding a placeholder (Mermaid treats a bare `{` as a diamond-node token even inside brackets).
6. Write to `.sdlc/wireframe.md`.

## Phase 7 — Approval loop

1. Ask the user to review `.sdlc/wireframe.md` and each `wireframes/<screen-slug>.html` (the last Artifact preview link works too) — approve or request changes, loop until approved. Changes to a specific screen → back to Phase 5 for that screen, then redo Phase 6 for it.
2. No git action follows approval — this command never branches, commits, or pushes.
3. Confirm `wireframe.md`'s path, the screen files, and that the published Artifact preview links are a session convenience only (may expire or be revoked) — the committed `.sdlc/wireframes/*.html` files are the durable output. Tell the user: optional next step `/codely.ui-template` (turns this into a high-fidelity design), or run `/clear` then `/codely.sys-init` and/or `/codely.backlog` directly.
