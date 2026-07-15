<!--
  Per-project config — edit before running /codely.spec on this project
  (or answer its first-run prompt to draft this file automatically).

  Used by /codely.spec to validate ticket IDs and build:
    branch:    <ticketId>/<slug>/feature
    directory: .specs/<ticketId>_<slug>/

  Replace the example below with this project's actual scheme.
-->

# Ticket ID Format

## Pattern

`PROJECT-NUMBER` — a JIRA-style key.

- **Regex:** `^[A-Z][A-Z0-9]{1,9}-\d+$`
- **Project key:** 2-10 uppercase letters/digits, starting with a letter (e.g. `PROJ`, `WEB3`)
- **Separator:** a single hyphen `-`
- **Issue number:** one or more digits, no leading zeros

## Valid examples

- `PROJ-123`
- `WEB-4821`

## Invalid examples

- `proj-123` — lowercase project key
- `PROJ_123` — underscore instead of hyphen
- `123` — missing project key
- `PROJ-` — missing issue number

## Case sensitivity

Case-sensitive. Always use the project key's canonical case (uppercase, as above).

## No ticket available

*(Optional — delete if this project always has a tracked ticket.)*

Use `NOTICKET-<yyyymmdd>` (e.g. `NOTICKET-20260715`) as a placeholder.