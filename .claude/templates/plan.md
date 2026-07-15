# Plan: {{Title}}

**Branch:** `{{ticketId}}/{{slug}}/feature`
**Ticket ID:** `{{ticketId}}`

## Approach

*Mandatory. High-level strategy and key technical decisions — how the spec's requirements will be implemented.*

{{approach}}

## Out of Scope

*Optional — omit if empty. Items explicitly confirmed out of scope during planning.*

{{out_of_scope}}

## Decisions

*Optional — omit if empty. Important points confirmed by the user during planning.*

{{decisions}}

## Phases

*Mandatory. Phases run in order — each phase's PR bases on `main` if the previous phase is already merged, otherwise stacks on the previous phase's branch.*

### Phase 1: {{phase_1_title}}

**Branch:** `{{ticketId}}/{{phase_1_slug}}`

#### Tasks

- [ ] {{task}} — done when {{criteria}}

#### Key Files

- {{file}}

<!-- Repeat "### Phase N: {{title}}" with its own Branch/Tasks/Key Files for each additional phase. -->

## QA Plan

*Mandatory. Checklist of test cases, ticked off by `/codely.qa`.*

- [ ] {{test_case}}