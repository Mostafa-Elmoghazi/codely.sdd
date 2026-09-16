<!--
  Shared pre-commit / branch-cut safety block.
  @-included at the top of the phase where a command, AFTER its approval gate, cuts the
  work branch off trunk and commits its drafted doc to it — /codely.spec, /codely.quick,
  /codely.bug-fix, /trainer.start. The command's own steps resume at 5 (create/checkout
  the branch, commit, push-or-not, confirm).
  Do not reorder or soften these steps — they are the guard against committing on the
  wrong branch or clobbering uncommitted work. Keep it in sync with any command that
  open-codes the same sequence (e.g. /codely.investigate, whose branch step is
  verdict-dependent and stays inline).
-->

**Don't start until this command's approval gate is explicit.** No git-mutating step runs before then.

1. `git status --porcelain`, disregarding the doc(s) this command just wrote in its drafting phase (and `.sdlc/backlog.md` if this run changed a ticket status in it) — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Trunk: `CLAUDE.md`'s choice, else `main`, else `master`. Ask if still ambiguous.
3. Zero commits (`git rev-parse --verify HEAD` fails): tell the user, skip switch/pull, ask whether to branch from the unborn state or commit first.
4. Otherwise: switch to trunk if needed; pull only if it tracks a remote. **Inside a ticket worktree** (see `.claude/includes/worktree-ticket.md`) never switch to trunk — fetch it (only if `origin` exists) and branch from `origin/<trunk>` — or local `<trunk>` when there's no `origin` or the fetch fails — per that file instead.
