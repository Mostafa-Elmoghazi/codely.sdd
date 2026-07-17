# Design: {{Title}}

**Branch:** `{{ticketId}}/{{slug}}/feature`
**Ticket ID:** `{{ticketId}}`

## Screens

*Mandatory. One entry per screen.*

### {{screen_title}}

**File:** `design/{{screen_slug}}.html`
**Platform:** {{mobile | website | desktop}}
**States covered:** {{states}}

{{screen_description}}

<!-- Repeat "### {{screen_title}}" with its own File/Platform/States for each additional screen. -->

## Component Breakdown

*Mandatory. Reusable (existing) vs. new components identified across the screens.*

{{component_breakdown}}

## Navigation / Flow

*Optional — omit if a single screen. How screens connect (entry points, transitions, back/cancel behavior).*

{{navigation_flow}}

## Non-Functional Notes

*Optional — omit if none. Responsive breakpoints, accessibility considerations, platform conventions followed.*

{{non_functional_notes}}

## Open Questions

*Optional — omit if none. Deferred to `follow-up.md` or resolved during `/codely.plan`.*

{{open_questions}}
