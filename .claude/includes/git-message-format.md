<!--
  Shared commit-message / PR-title format for ticket work.
  @-included, right after no-ai-attribution.md, by every ticket command that commits or opens a
  PR: /codely.spec, /codely.quick, /codely.bug-fix, /codely.investigate, /codely.plan,
  /codely.sys-design, /codely.ui, /codely.implement, /codely.qa, /codely.follow-up,
  /codely.address-review. The messages each command names are written in this format already.
-->

**Commit and PR naming.** Every commit this command makes for a ticket uses `<ticketId>: <message>` — e.g. `PROJ-12: Add spec`. Every PR it opens uses:

- **A phase of a multi-phase ticket** (`/codely.implement`): `<ticketId>: <phase number> - <phase title>` — e.g. `PROJ-12: 2 - Add export endpoint`. The phase number is the phase's number in `plan.md`.
- **Any other ticket PR**: `<ticketId>: <title text>` — e.g. `PROJ-12: Fix login timeout`.

A placeholder ID (`NOTICKET-<date>`) counts as the ticket ID. No ticket ID at all → plain `<message>` / `<title text>`. A `CLAUDE.md` rule for commit or PR naming overrides this one.
