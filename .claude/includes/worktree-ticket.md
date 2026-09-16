<!--
  Shared ticket-worktree rules.
  A ticket worktree is one created by /codely.worktree: a linked git worktree at
  `<repo root>/.claude/worktrees/<ticketId>_<slug>/`, checked out on the ticket's
  `<ticketId>/<slug>/feature` branch. The directory name is the source of truth for
  ticketId + slug, so commands run inside it need no ticket ID argument.
  @-included in Phase 0, right after resolve-ticket-inputs.md, by /codely.spec,
  /codely.quick, /codely.bug-fix, /codely.investigate. Referenced by path from
  /codely.plan, /codely.implement, /codely.ui, /codely.sys-design, /codely.qa,
  /codely.follow-up, and .claude/includes/git-preflight.md.
-->

**Ticket worktree detection.** You're in a ticket worktree when `git rev-parse --git-dir` differs from `git rev-parse --git-common-dir` (a linked worktree) **and** the basename of `git rev-parse --show-toplevel` is `<ticketId>_<slug>` with a parent directory named `.claude/worktrees`, where `ticketId` validates against `.claude/includes/ticket-format.md`. Split the basename at the first `_` whose left part validates.

In a ticket worktree:

- **Ticket ID + slug come from the directory name.** Use them whenever `$ARGUMENTS` doesn't supply a ticket ID — never ask the user for one. If `$ARGUMENTS`' first token doesn't validate as a ticket ID, treat all of `$ARGUMENTS` as the description/question instead. If it validates but differs from the worktree's ticket → warn, AskUserQuestion (use the worktree's ticket / use the typed one / cancel).
- **The slug is fixed.** Reuse the worktree's slug; skip slug generation. An existing `.specs/<ticketId>_*` with a different slug → tell the user, and prefer the `.specs/` slug (it's what the docs already use).
- **Trunk can't be checked out here** — it's checked out in the main worktree. Wherever a step says "switch to trunk" / "pull trunk", instead `git fetch origin <trunk>` (only if `origin` exists) and use `origin/<trunk>` (else local `<trunk>`) as the base ref for any new branch. Never `git switch <trunk>` inside a worktree.
- **Creating a branch other than the ticket's `/feature` branch** (e.g. `/quick`, `/bug-fix`): current branch is `<ticketId>/<slug>/feature` with no commits ahead of the base ref and no upstream → rename it in place (`git branch -m <new branch>`) instead of leaving an empty `/feature` branch behind. Otherwise create the new branch from the base ref with `git switch -c <new branch> <base ref> --no-track`.
