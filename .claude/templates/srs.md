# Software Requirements Specification: {{Title}}

**Charter:** `.sdlc/project-charter.md`
**Use-Case Model:** `.sdlc/use-case-model.md`

## Status

*Tracks interview progress across sessions so work can pause and resume without redoing settled topics. Check off each topic as it's resolved — safe to ignore once every box is checked. If a topic turns out to be multi-part (spans several question rounds or sessions), split it into indented sub-items and check those off individually; check the parent once all its sub-items (or the topic as a whole) are settled. Don't delete this section. Purpose & Scope and Traceability aren't tracked here — they're synthesized from the charter and from the topics below, not interviewed separately.*

- [ ] Functional Requirements
- [ ] Non-Functional Requirements
- [ ] Validation

## Purpose & Scope

*Mandatory. What this SRS covers, and how it traces back to the charter's goals and scope.*

{{purpose_scope}}

## Functional Requirements

*Mandatory. Specific, testable behaviors the system must implement, grouped by feature area or use case.*

### {{requirement_group}}

- **FR-{{n}}:** {{requirement}}

<!-- Repeat "### {{requirement_group}}" for each additional group. -->

## Non-Functional Requirements

*Mandatory. Performance, security, compliance, accessibility, scalability, etc.*

- **NFR-{{n}}:** {{requirement}}

## Assumptions & Constraints

*Optional — omit if none.*

{{assumptions_constraints}}

## Traceability

*Mandatory. Map requirements back to the charter's goals, so nothing drifts from why the project exists.*

| Requirement | Charter Goal |
|---|---|
| {{requirement_id}} | {{goal_reference}} |

## Decisions

*Optional — omit if empty. "You decide" calls made during the interview, with rationale.*

{{decisions}}
