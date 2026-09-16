<!--
  Shared "Resolve inputs" block.
  @-included as the body of Phase 0 by commands that take `[ticket-id] [free text]`
  arguments and start work on a ticket — /codely.spec, /codely.quick, /codely.bug-fix,
  /codely.investigate.
  The including command states, right above the include, what its **input text** is
  (e.g. "the brief description", "the bug description", "the question / proposed work");
  every mention of "input text" below means that.
  Also @-include .claude/includes/worktree-ticket.md and .claude/includes/ticket-source.md
  right after this block — their rules are referenced in steps 2-3.
-->

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: first token (minus trailing `:`) = ticket ID, rest = input text. Inside a ticket worktree, the rules in `.claude/includes/worktree-ticket.md` override this and step 4 — the ticket ID comes from the worktree, never from asking.
3. Ticket ID present → resolve the ticket from its source, per `.claude/includes/ticket-source.md` (tracker or `.sdlc/backlog.md`; a tracker whose MCP connection is missing is reported and skipped — carry on with the typed input text, or ask for it).
   - Input text empty → use the ticket's title (plus a short lead from its details if that helps) as the input text; tell the user what was fetched and from where before proceeding — never substitute it silently.
   - Input text given → keep it; use the ticket's details as extra context only.
   - Source is the backlog → mark the row `in progress` per `ticket-source.md`.
4. Still missing the ticket ID and/or input text: ask the user for whatever's missing, then run step 3 for a newly given ticket ID. Don't continue without both.
