---
description: Fetch a PR's unresolved review threads, address each one, then reply and resolve
argument-hint: [ticket-id]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.address-review

> **Scope guardrail:** acts only on the current PR's unresolved review threads — each gets a code fix, a reply-only answer, or a reasoned rebuttal, nothing broader. Within `.specs/<ticketId>_<slug>/`, touches only `follow-up.md` if new deferred work surfaces. One commit + push to the PR branch, one reply per thread, and a resolve on every thread that was fixed or answered — threads it disagrees with are replied to but left unresolved.

Fetch every unresolved review thread on the PR for the current branch and clear it — in one pass, **no chat approval gate**. Built to run right after a PR is opened and the first review comments land. Re-running is safe: resolved threads don't come back. `CLAUDE.md` overrides this file where they conflict.

**Plain-language standard** — applies to every reply this command posts:

@.claude/includes/clarity.md

## Context

- Current branch: !`git branch --show-current 2>/dev/null || echo "(none)"`
- Working tree: !`git status --porcelain 2>/dev/null || echo "(not a git repo)"`
- `gh` ready: !`gh auth status >/dev/null 2>&1 && echo yes || echo no`

## Steps

1. **Resolve the PR.** Read `CLAUDE.md`. `gh` unavailable/unauthenticated → stop (threads can't be listed or resolved without it). Derive `ticketId` from the branch (`<ticketId>/...`) or `$ARGUMENTS`, else ask. Find the open PR for the current branch — none → stop, tell the user to open one. Not on the PR's head branch, or a dirty tree → AskUserQuestion, don't auto-act.
2. **Load context.** Read the PR's unresolved review threads. Then read `.specs/<ticketId>_*`'s `spec.md` / `system-design.md` / `design.md` / plan / `follow-up.md` where present, plus the code each thread points at — enough to judge each comment against what was approved. Nothing unresolved → stop.
3. **Triage each thread**, then report the table (thread → disposition) as a heads-up and continue:
   - **Fix** — valid and consistent with the approved spec/design; a scoped code change addresses it.
   - **Reply only** — a question or already handled; needs an answer, not code.
   - **Disagree** — wrong or conflicts with the approved spec/design; reply with reasoning, leave unresolved.
   - **Needs your call** — turns on a material decision the spec doesn't settle → AskUserQuestion, then re-classify.
4. **Fix.** Implement the Fix threads — minimal, scoped to the comment, matching local conventions. Necessary work outside any comment's scope → append to `follow-up.md` (from `.claude/templates/follow-up.md`) and flag it, don't do it silently.
5. **Verify** (skip if no code changed): run the project's build/lint/test (`CLAUDE.md`, then `package.json`/`Makefile`). Fix failures and re-run.
6. **Ship.** If code changed: one commit `Address review comments for <ticketId>`, push. Reply to every thread (Fix → what changed + commit SHA; Reply only → the answer; Disagree → the reasoning). Resolve every Fix and Reply-only thread; leave Disagree threads unresolved. Confirm: counts by disposition, commit SHA, PR URL, threads left unresolved and why.
7. **Next.** Unresolved threads → follow up with the reviewer, re-run this after they reply. New `follow-up.md` items → `/codely.follow-up`. Code changed after `/codely.qa` already ran → re-run it.
