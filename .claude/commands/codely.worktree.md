---
description: Create a ready-to-use git worktree + feature branch for a ticket
argument-hint: <ticket-id>
allowed-tools: Read, Glob, Grep, Write, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.worktree

> **Scope guardrail:** never writes application code, specs, or plans, and never commits or pushes. Outputs are a linked git worktree at `.claude/worktrees/<ticketId>_<slug>/` on a new local `<ticketId>/<slug>/feature` branch, a `safe.directory` entry for it in the user's global git config (only when git requires one), copies of the main worktree's local `.env` files, and installed dependencies. Everything else in the main worktree is left untouched — including uncommitted changes and the checked-out branch.

Turn a ticket ID into an isolated, ready-to-work worktree, so the user can `cd` into it and run any `codely.*` command (`/codely.spec`, `/codely.quick`, `/codely.bug-fix`, `/codely.investigate`, …) without typing the ticket ID — those commands read it from the worktree's directory name. Follow the phases in order. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks:

@.claude/includes/clarity.md

## Context

- Main worktree root: !`dirname "$(cd "$(git rev-parse --git-common-dir 2>/dev/null)" && pwd)" 2>/dev/null || echo "(not a git repository)"`
- Current worktree root: !`git rev-parse --show-toplevel 2>/dev/null || echo "(none)"`
- Existing worktrees: !`git worktree list 2>/dev/null || echo "(none)"`
- `.claude/worktrees/` contents: !`ls -1 "$(dirname "$(cd "$(git rev-parse --git-common-dir 2>/dev/null)" && pwd)")/.claude/worktrees" 2>/dev/null || echo "(empty or missing)"`
- Remotes: !`git remote 2>/dev/null || echo "(none)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`
- Task tracker config present: !`test -f .claude/includes/task-tracker.md && echo yes || echo no`

## Phase 0 — Resolve the ticket

1. Read `CLAUDE.md` if present.
2. All paths below are relative to the **main worktree root** (from Context), even when this command is run from inside another worktree. Call it `ROOT`.
3. Ticket ID = `$ARGUMENTS`' first token (minus trailing `:`). Empty → ask the user. Don't guess.
4. Read `.claude/includes/ticket-format.md` and validate the ID against it. No match → show the expected format, AskUserQuestion — corrected ID / cancel. (Missing config → stop and tell the user to run `/codely.spec` once to set it up.)

## Phase 1 — Look up the ticket

Resolve the ticket from its source (reading `ROOT/.claude/includes/task-tracker.md` and `ROOT/.sdlc/backlog.md`). This command only looks the ticket up — it never changes a backlog status; `/codely.spec`, `/codely.quick`, etc. do that when work starts.

@.claude/includes/ticket-source.md

Found → show the user the ticket ID, title, and where it came from (the tracker's item key + state, or the backlog row + status). The title drives the slug.

## Phase 2 — Derive the slug

Reuse an existing slug for this ticket if there is one — first match wins:

1. An existing worktree directory `ROOT/.claude/worktrees/<ticketId>_*` → go straight to Phase 3 step 1.
2. `ROOT/.specs/<ticketId>_*` → its slug.
3. A local or remote branch `<ticketId>/<slug>/<kind>` (`git branch -a --list "*<ticketId>/*"`; exact ticket ID match only — `PROJ-1` must not match `PROJ-10`; ignore two-segment phase branches `<ticketId>/<phaseSlug>`, whose slug is a phase's, not the ticket's) → its slug. Several different slugs → AskUserQuestion.
4. Otherwise generate it from the ticket title:

@.claude/includes/slug.md

Worktree name = `<ticketId>_<slug>`; path = `ROOT/.claude/worktrees/<ticketId>_<slug>`.

## Phase 3 — Guard against duplicates

1. That path (or any `<ticketId>_*` sibling) already exists and is listed by `git worktree list` → don't recreate it. Run Phases 5 and 6 (both idempotent) so it's still marked safe and a failed env-file copy or dependency install from an earlier run gets retried, then go to Phase 7 and report it as already existing, with its current branch.
2. Directory exists but isn't a registered worktree (stale leftover) → tell the user, AskUserQuestion — remove the directory and continue / cancel. Never delete without confirmation. If `git worktree list --porcelain` shows it as `prunable`, run `git worktree prune` first.
3. `git -C ROOT check-ignore -q .claude/worktrees/` fails (not ignored) → tell the user worktrees there would show up as untracked files in the main repo, and AskUserQuestion — add `.claude/worktrees/` to `ROOT/.gitignore` (left uncommitted for them to commit) / cancel.

## Phase 4 — Create the worktree and branch

1. Trunk: `CLAUDE.md`'s choice, else `main`, else `master`.
2. `origin` exists → `git -C ROOT fetch origin <trunk>`; base ref = `origin/<trunk>`. No remote or fetch fails → base ref = local `<trunk>` (tell the user if the fetch failed).
3. Pick the branch:
   - A local branch for this ticket + slug already exists (`<ticketId>/<slug>/feature`, `/quick`, `/bug-fix`, or `/investigation`; more than one → AskUserQuestion) → reuse it. If it's already checked out in another worktree → stop and report where. Otherwise `git -C ROOT worktree add <path> <branch>`, and note how far ahead of the base ref it is.
   - Only a remote branch exists → `git -C ROOT worktree add --track -b <branch> <path> origin/<branch>`.
   - None → `git -C ROOT worktree add --no-track -b <ticketId>/<slug>/feature <path> <base ref>`.
4. Never commit, never push, never touch the main worktree's checked-out branch or working tree.

## Phase 5 — Mark the directory safe (when git needs it)

Repositories on drives owned by another user (external/mounted drives, shared volumes) make git refuse to work in each new directory until it's listed in `safe.directory`.

1. Absolute path = `realpath <path>`.
2. `git -C "<abs path>" status --porcelain` succeeds → nothing to do; report "not needed".
3. Fails with a "dubious ownership" error → `git config --global --get-all safe.directory | grep -Fx -- "<abs path>"`; not already present → `git config --global --add safe.directory "<abs path>"`. Re-run the status check to verify.

## Phase 6 — Make it ready to work

1. **Local env files.** Find the main worktree's git-ignored env files: `find ROOT -name node_modules -prune -o -name .claude -prune -o -name .git -prune -o -type f \( -name .env -o -name .env.local \) -print`, keeping only those `git -C ROOT check-ignore -q` confirms are ignored. For each one whose counterpart in the worktree is missing, copy it to the same relative path. Never overwrite an existing file; never copy `.env.example` (it's tracked). List what was copied.
2. **Dependencies.** Install them in the worktree with the command `CLAUDE.md` names; otherwise pick by lockfile — `pnpm-lock.yaml` → `pnpm install --frozen-lockfile`, `package-lock.json` → `npm ci`, `yarn.lock` → `yarn install --frozen-lockfile`, `bun.lockb` / `bun.lock` → `bun install --frozen-lockfile`, other ecosystems' lockfiles → their equivalent frozen install; nothing recognisable → skip and say so. Fails → report the error and carry on; the worktree is still usable for doc-only commands (`/codely.spec`, `/codely.investigate`).
3. Don't start shared services (Docker, databases) or run migrations — local infra is shared with the main worktree.

## Phase 7 — Confirm

Report:

- Ticket: ID + title, and its source (the tracker's item key, or `.sdlc/backlog.md`).
- Worktree path (absolute) and branch, and the base ref it was cut from (or "reused existing branch, N commits ahead").
- `safe.directory`: added / already present / not needed.
- Env files copied, and dependency install result.

Then tell the user how to start:

```
cd <abs path> && claude
```

…and that inside it they can run `/codely.spec`, `/codely.quick`, `/codely.bug-fix`, or `/codely.investigate` with no ticket ID — the ticket is read from the worktree name. When the ticket is merged, clean up from the main worktree with `git worktree remove .claude/worktrees/<ticketId>_<slug>`.
