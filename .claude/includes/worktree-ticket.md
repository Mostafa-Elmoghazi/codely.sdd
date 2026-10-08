<!--
  Shared ticket-worktree rules.
  A ticket worktree is a linked git worktree at `<repo root>/.claude/worktrees/<ticketId>/` or
  `<repo root>/.claude/worktrees/<ticketId>_<slug>/` (the form /codely.worktree creates), checked
  out on one of the ticket's branches (`<ticketId>/<slug>/feature` at first). The directory name
  is the source of truth for the ticketId (and the slug, when it carries one), so commands run
  inside it need no ticket ID argument.
  @-included as the FIRST step of Phase 0 — before any branch, `.specs/`, or CLAUDE.md read —
  by every ticket command: /codely.spec, /codely.quick, /codely.bug-fix, /codely.investigate,
  /codely.plan, /codely.sys-design, /codely.ui, /codely.implement, /codely.qa,
  /codely.follow-up, /codely.address-review. Referenced by path from /codely.compact,
  .claude/includes/resolve-ticket-inputs.md, and .claude/includes/git-preflight.md.
-->

**Ticket worktrees.** `ROOT` = the main project folder: `dirname "$(cd "$(git rev-parse --git-common-dir)" && pwd)"` (the same from any worktree). A **ticket worktree** is an entry in `git worktree list --porcelain` (skip `prunable` ones) whose path is `ROOT/.claude/worktrees/<name>`, where `<name>` is either `<ticketId>` or `<ticketId>_<slug>` and `ticketId` validates against `.claude/includes/ticket-format.md` — the whole basename validates → it's the bare ticket ID (no slug in the name); otherwise split it at the first `_` whose left part validates. A worktree **matches** a ticket when its directory name equals `<ticketId>` or starts with `<ticketId>_`, or its checked-out branch starts with `<ticketId>/` — exact ID only (`PROJ-1` never matches `PROJ-10`). `ticket-format.md` missing → no worktree can match; skip straight to the command's own steps.

**Step 0 — Pick where to work.** Do this before anything else in Phase 0, because the Context above was gathered in the folder the session started in, which may not be where this ticket's work lives.

1. **Find the ticket asked for.** `$ARGUMENTS`' first token (minus trailing `:`) if it validates as a ticket ID → the **typed ticket**. Otherwise the session folder's current branch starts with `<ticketId>/` for a valid ID → the **branch ticket**. Otherwise → no ticket yet.
2. **Session is already in a ticket worktree** (`git rev-parse --git-dir` ≠ `git rev-parse --git-common-dir` and `git rev-parse --show-toplevel` is a ticket worktree path):
   - No typed ticket, or the typed ticket is this worktree's → stay here.
   - The typed ticket differs and **has its own worktree** → tell the user in one line and switch into it (step 4).
   - The typed ticket differs and **has no worktree** → its work belongs in the main project folder. Tell the user, AskUserQuestion — go back to the main folder and continue there (Recommended) / use this worktree's ticket instead / cancel. Going back: `ExitWorktree` with `action: "keep"` (never `remove`). That's a no-op when Claude was started inside this worktree → tell the user to run the command from the main folder (`cd ROOT && claude`), or to create a worktree first with `/codely.worktree <ticketId>`, and stop.
3. **Session is in the main project folder:**
   - The typed ticket (else the branch ticket) has a matching ticket worktree → tell the user in one line that this ticket has a worktree and the work continues there, then switch into it (step 4). More than one matches → AskUserQuestion which.
   - No ticket yet, and ticket worktrees exist → AskUserQuestion — "Which ticket is this for?": each ticket worktree as an option (ticket ID + its branch; the one with the most recent commit first, marked Recommended), plus "Not one of these — work in the main folder". Over 3 worktrees → list the 3 most recent; the free-text answer covers the rest. A worktree picked → switch into it (step 4). The main folder picked → carry on with the command's own steps here.
   - Otherwise → stay in the main folder and carry on with the command's own steps here.
4. **Switching into a worktree:** call `EnterWorktree` with `path` = the worktree's absolute path. It moves the whole session there: every git command, file read/write, `.specs/` lookup, `CLAUDE.md` read, build, and test from here on runs inside the worktree. The main folder is left untouched — never check its working tree for dirtiness or switch its branch. Then re-gather the Context values (working tree status, current branch, `.specs/` contents, …) in the worktree — the ones above describe the folder you left. `EnterWorktree` unavailable or refused → tell the user to run `cd <worktree path> && claude` and re-run the command there; stop.

After Step 0, the rest of this command runs in the chosen folder. A typed ticket ID always beats the one in the current branch — if they differ, say so before going on. Git refuses to switch to a branch that's checked out in another worktree → tell the user which folder has it, and AskUserQuestion — continue in that worktree (switch per step 4; Recommended) / cancel.

**In a ticket worktree:**

- **Ticket ID comes from the directory name.** Use them whenever `$ARGUMENTS` doesn't supply a ticket ID — never ask the user for one. If `$ARGUMENTS`' first token doesn't validate as a ticket ID, treat all of `$ARGUMENTS` as the description/question (or, for `/codely.implement`, the phase number) instead. A typed ID that differs was already settled in Step 0.
- **The worktree's slug**, first match wins: an existing `.specs/<ticketId>_*` directory's slug (it's what the docs already use — if the directory name carries a different slug, tell the user) → the slug in the directory name (`<ticketId>_<slug>`) → the slug in the checked-out branch (`<ticketId>/<slug>/<kind>`; a two-segment phase branch `<ticketId>/<phaseSlug>` carries a phase's slug, not the ticket's — ignore it). Found → the slug is known and fixed: reuse it and skip slug generation. None (a bare `<ticketId>` folder on a fresh branch) → the slug isn't known yet; generate it as the command normally would.
- **Trunk can't be checked out here** — it's checked out in the main worktree. Wherever a step says "switch to trunk" / "pull trunk", instead `git fetch origin <trunk>` (only if `origin` exists) and use `origin/<trunk>` (else local `<trunk>`) as the base ref for any new branch. Never `git switch <trunk>` inside a worktree.
- **Creating a branch other than the ticket's `/feature` branch** (e.g. `/quick`, `/bug-fix`): current branch is `<ticketId>/<slug>/feature` with no commits ahead of the base ref and no upstream → rename it in place (`git branch -m <new branch>`) instead of leaving an empty `/feature` branch behind. Otherwise create the new branch from the base ref with `git switch -c <new branch> <base ref> --no-track`.
