# Ticket Backlog: {{Title}}

**SRS:** `.sdlc/srs.md`
**HLD:** `.sdlc/hld.md`
**LLD:** `.sdlc/lld.md`

## How to use this

*This is a proposed, board-agnostic breakdown of the approved design into implementation-sized tickets — not a live sync with any dev board. Create each ticket on your team's actual tracker (Jira/Linear/GitHub Issues/etc.), then run `/codely.spec <ticketId> <description>` to start it, in the sequence below. The Board ID column is for your own reference only — `/codely.spec` doesn't read this file, so nothing here needs to stay in sync with the real tracker.*

## Wave 1: {{wave_1_title}}

*Mandatory — at least one wave. Tickets within a wave have no dependencies on each other; later waves depend on earlier ones.*

| Proposed Title | Description | Source | Depends On | Size | Board ID |
|---|---|---|---|---|---|
| {{ticket_title}} | {{one_line_description}} | {{fr_or_component_refs}} | {{depends_on}} | {{size}} | |

<!-- Repeat "## Wave N: {{title}}" with its own table for each additional wave. -->

## Out of Scope / Deferred

*Optional — omit if empty. Candidate work identified but deliberately excluded from this backlog (e.g. explicitly out of scope per the charter, or deferred to a later phase of the project plan).*

{{deferred_items}}

## Decisions

*Optional — omit if empty. "You decide" calls made during the review, with rationale (sizing ties, sequencing calls, slicing choices).*

{{decisions}}
