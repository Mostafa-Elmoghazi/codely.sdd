# Ticket Backlog: {{Title}}

**SRS:** `.sdlc/srs.md`
**HLD:** `.sdlc/hld.md`
**LLD:** `.sdlc/lld.md`

## How to use this

*A breakdown of the approved design into implementation-sized tickets, each with a ticket ID that follows this project's `.claude/includes/ticket-format.md` scheme, in the sequence below. **With a task tracker** configured in `.claude/includes/task-tracker.md`: create each ticket there using the ID shown — the tracker is the source of truth, and the Status column here isn't updated. **Without one** (`Not used for this repo`): this file is the tracker — run `/codely.spec <ticketId>` or `/codely.quick <ticketId>` and the ticket's title and description are read from here. Status moves `to do` → `in progress` when a command starts the ticket, and → `done` when `/codely.qa` / `/codely.follow-up` finish it.*

## Wave 1: {{wave_1_title}}

| Ticket ID | Title | Description | Source | Depends On | Size | Status |
|---|---|---|---|---|---|---|
| {{ticket_id}} | {{ticket_title}} | {{one_line_description}} | {{fr_or_component_refs}} | {{depends_on}} | {{size}} | to do |

## Out of Scope / Deferred

{{deferred_items}}

## Decisions

{{decisions}}
