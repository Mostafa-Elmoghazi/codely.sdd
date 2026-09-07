---
description: Turn the approved system design (and wireframe, if present) into a high-fidelity, whole-product HTML/CSS design system
argument-hint: [optional focus area or notes]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Artifact
disable-model-invocation: true
---

# codely.ui-template

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.sdlc/ui-template.md` and `.sdlc/ui-template/`. This command makes no git changes (no branch, no commit, no PR) — approval happens in chat. Published Artifact preview links are a live-iteration convenience for this session, not the durable output — the committed `.sdlc/ui-template/*.html` files are.

**Optional step.** Turn the approved system design — and `.sdlc/wireframe.md`, if one exists — into a high-fidelity, whole-product HTML/CSS design system at `.sdlc/ui-template.md` + `.sdlc/ui-template/<screen-slug>.html`: real design tokens (palette, typography, spacing), applied across the key screens. Skip this command if the project already has an established in-repo design system, or branding will be handled separately. This is the counterpart to `/codely.ui` at project scope — its output is exactly what `/codely.ui`'s design-system discovery step looks for first, so every later ticket's mockups share one look instead of each reinventing a placeholder style. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Phase 0 — Load context

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md`. **Missing either:** tell the user this needs an approved system design first, stop, recommend `/sdlc.system-design`.
3. Read both — the HLD's UI/UX Overview bounds the screen set if no wireframe exists yet.
4. Glob `.sdlc/wireframe.md` — **present:** read it, use its screens and navigation as the structural basis for this pass. **Absent:** derive the screen/layout list directly from the HLD's UI/UX Overview, and tell the user this pass is deciding structure as well as visuals since no wireframe exists — `/codely.wireframe` remains available to run first if they'd rather settle structure separately.
5. Glob `.sdlc/project-charter.md` — present → read it too, its Goals/Scope carry the project idea, industry, and target audience that ground Phase 3's theme proposal.
6. Glob `.sdlc/srs.md` — present → read it too, for non-functional constraints (accessibility, contrast, branding requirements) that bound the design tokens.
7. `$ARGUMENTS` non-empty: treat it as extra focus for this pass.

## Phase 1 — Check for an existing template

1. Glob `.sdlc/ui-template.md`.
2. **Missing:** proceed to Phase 2.
3. **Found:** read and summarize its theme direction and screens. AskUserQuestion — Resume/revise (add or revise screens, keep the theme) / Start over (regenerate all) / Cancel.

## Phase 2 — Design system discovery

Glob/Grep/Read the codebase for existing design conventions — CSS variables/tokens, Tailwind or similar config, theme files, a shared component library, existing screens' markup and styling, brand colors/fonts, any style guide or Storybook. Found → this pass formalizes and extends what's real; if Phase 3's interview points somewhere that conflicts with it, flag the conflict to the user rather than silently overriding it. Nothing found → proceed fresh in Phase 3.

## Phase 3 — Theme direction (interview)

**Mandatory — don't start drafting until a theme direction and its concrete tokens are confirmed.**

1. Synthesize the signals already on disk: the charter's stated idea/pitch and Goals/Scope, the industry/domain it serves, the target audience/persona (charter plus the use-case model's actors, if read), and the target platform(s).
2. Propose 2-3 candidate visual directions from that synthesis — not an open-ended "what should it look like." Each candidate: a short mood/tone label, a palette direction, a typography pairing, and a one-line rationale tying it back to the industry/audience signals (e.g. "enterprise treasury dashboard for finance ops teams → 'Restrained & precise': low-saturation blues/neutrals, high information density, a grounded serif for numerals — reads trustworthy and reduces visual noise for long sessions"). Anything found in Phase 2 becomes (or constrains) one of these candidates instead of a fresh proposal.
3. AskUserQuestion (previews where useful) — pick a candidate, tweak it, or "you decide."
4. On "you decide," state one concrete recommendation and get explicit confirmation — record it under Decisions.
5. Follow-up rounds (cap 4/call) to pin concrete tokens: primary/secondary/accent colors, font families, spacing/radius scale, tone (playful/serious/dense/spacious), dark-mode and contrast constraints.
6. Digest each round; run another if new questions surface.

## Phase 4 — Draft and preview each screen

For each screen (from `.sdlc/wireframe.md` if present, else Phase 0's HLD-derived list):

1. Build a self-contained, high-fidelity HTML/CSS template: real representative content (not lorem ipsum), the confirmed design tokens applied as real CSS custom properties, and the states the wireframe (or HLD) identifies for that screen.
2. Wrap per platform: mobile → phone-frame viewport; desktop → window-chrome frame; website → plain responsive layout, no wrapper.
3. Publish via the Artifact tool and share the URL.
4. Walk the user through it — layout, tokens applied, content, states, interactions. Revise and republish to the same URL each round until the user confirms this screen. Move to the next screen once confirmed.
5. A screen turns out to need splitting, merging, or dropping mid-draft → tell the user before continuing (and update the wireframe reference if one exists — this command never edits `wireframe.md` itself, just notes the drift).

## Phase 5 — Write the template files

1. Write the confirmed design tokens as real CSS custom properties to `.sdlc/ui-template/tokens.css`.
2. For each confirmed screen, save its final HTML to `.sdlc/ui-template/<screen-slug>.html` (self-contained, matching the last published Artifact revision, referencing `tokens.css`'s values).
3. Fill `.claude/templates/ui-template.md` placeholders.
4. Mandatory: Theme Direction (the confirmed candidate and its rationale from Phase 3), Design Tokens (a table of every token written to `tokens.css` — name, value, usage), Screens (one entry per screen — repeat `### {{screen_title}}` with its own File/Platform/States for each additional screen).
5. Optional (omit a section entirely when empty): Component Breakdown (reusable UI components identified across the screens), Open Questions (deferred to ticket-level `/codely.ui` work).
6. Write to `.sdlc/ui-template.md`.

## Phase 6 — Approval loop

1. Ask the user to review `.sdlc/ui-template.md`, `tokens.css`, and each `ui-template/<screen-slug>.html` (the last Artifact preview link works too) — approve or request changes, loop until approved. Changes to a specific screen → back to Phase 4 for that screen, then redo Phase 5 for it.
2. No git action follows approval — this command never branches, commits, or pushes.
3. Confirm `ui-template.md`'s path, the token and screen files, and that the published Artifact preview links are a session convenience only (may expire or be revoked) — the committed `.sdlc/ui-template/*.html` files are the durable output. Tell the user: this becomes the design system `/codely.ui` will discover and reuse for every ticket; next steps `/codely.sys-init` and/or `/codely.backlog`.
