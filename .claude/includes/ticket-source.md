<!--
  Shared "where do this repo's tickets live" rules.
  @-included by commands that look a ticket up or change its status:
    - resolve-ticket-inputs.md (/codely.spec, /codely.quick, /codely.bug-fix,
      /codely.investigate) — look up + mark `in progress`
    - /codely.worktree — look up only
    - /codely.qa, /codely.follow-up — mark `done`
  The tracker setting itself lives in .claude/includes/task-tracker.md (per project,
  written on first use — see step 1).
-->

**Ticket source.** Every ticket comes from exactly one source — never mix them:

1. **Determine the source** from `.claude/includes/task-tracker.md`:
   - Names a tracker (tracker name + project) → **tracker**.
   - Says `Not used for this repo` → **backlog** (`.sdlc/backlog.md`).
   - **Missing** → AskUserQuestion once — "Where do this repo's tickets live?": each task-tracking MCP connector available in this session (Plane, Jira/Atlassian, Linear, GitHub Issues, Asana, …) as an option, plus `.sdlc/backlog.md`. (A tracker whose connector isn't connected right now can still be named via the free-text answer.)
     - **A tracker** → through its MCP tools, list its projects; exactly one matches this repo → use it, else AskUserQuestion to pick. Then work out how this repo's ticket IDs (per `.claude/includes/ticket-format.md`) map to that tracker's items — the tracker's own key, or a separate field such as an external ID or custom field; check one existing item if any exist, otherwise ask. Write `.claude/includes/task-tracker.md`, mirroring `ticket-format.md`'s header-comment style, with sections: **Tracker** (name + MCP server), **Project** (name + ID), **Ticket ID lookup** (the ordered MCP calls that find an item by ticket ID).
     - **`.sdlc/backlog.md`** → write `.claude/includes/task-tracker.md` with the body `Not used for this repo — tickets are tracked in .sdlc/backlog.md.`
     - Either way, tell the user to commit it — it's shared project config.

2. **Tracker source:**
   - The tracker's MCP tools aren't available in this session (not connected, not authorized, or erroring on connect) → tell the user plainly in one line: the `<tracker>` MCP connection isn't available, so the ticket can't be looked up. **Carry on without it:** use the description the user typed; none typed → ask the user for it (a title is enough where the command only needs one). Never fall back to `.sdlc/backlog.md`, even if it lists the ticket.
   - Available → look the ticket up following `task-tracker.md`'s **Ticket ID lookup** steps, in order. Found → its title is the ticket's title, its description the details. Not found → tell the user, AskUserQuestion — corrected ticket ID / cancel.
   - Never read or write `.sdlc/backlog.md` statuses on this path.

3. **Backlog source:**
   - `.sdlc/backlog.md` missing → tell the user there's no backlog to take tickets from; recommend `/codely.backlog`. Stop.
   - Find the table row whose **Ticket ID** cell equals the ticket ID exactly (`PROJ-1` never matches `PROJ-10`). Its Title and Description are the ticket's title and details. The backlog uses the legacy shape (IDs inside titles, no Ticket ID column) → tell the user to run `/codely.backlog` and pick Refresh to update it first. Stop.
   - Not found → tell the user, AskUserQuestion — corrected ticket ID / cancel.

4. **Status updates — backlog source only.** Statuses are `to do`, `in progress`, `done`. To set one, edit only that row's **Status** cell in `.sdlc/backlog.md`; leave every other cell and row untouched. A table with no Status column → add it as the last column to every wave table, filling `to do`, then set this row. Placeholder IDs (`NOTICKET-<date>` / no-ticket forms) aren't unique → skip status updates for them.
   - **Mark `in progress`** (commands that start a ticket): when the row isn't already `in progress` or `done`, set it right after the ticket is resolved and tell the user in one line. The row says `to do` but a local or remote branch `<ticketId>/…` already exists (other than the one this run is on) → the ticket was likely started elsewhere and its status just hasn't reached this branch yet (statuses travel with each ticket's branch and reach trunk on merge); tell the user which branch, and AskUserQuestion — continue here / cancel. A `done` row → tell the user and AskUserQuestion — reopen it as `in progress` / leave it `done`. The command stops or is cancelled before its first commit → put the row back to what it was.
   - **Mark `done`** (completion commands): only when the command's own completion condition is met; tell the user in one line. Completion commands only update status — they never look the ticket up, never ask where tickets live, and never stop over the tracker: `task-tracker.md` missing or naming a tracker (connected or not) → skip the status update silently. Only `Not used for this repo` + a matching backlog row → set `done`.
   - **Committing:** `.sdlc/backlog.md` counts as one of this command's own files — ignore it when checking for a dirty working tree, and stage it with the command's next commit (inside a worktree, that commit is on the ticket's branch, so trunk sees the new status when the PR merges).
