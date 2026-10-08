---
description: Summarize this conversation to a file so you can /clear and keep going with a cheap, fresh context
argument-hint: (none)
allowed-tools: Read, Glob, Write, Bash(git branch --show-current), Bash(git worktree list:*), Bash(git rev-parse:*), Bash(date:*)
disable-model-invocation: true
---

# codely.compact

> **Scope guardrail:** writes only a session summary file — never application code, never `spec.md`/`plan.md`/any other ticket artifact, never git state. This command cannot clear the conversation itself: `/clear` and `/compact` are CLI-level actions with no tool-call equivalent exposed to the model, so the last step is always an instruction to the user, never an automatic reset.

Summarize the entire conversation so far into one file, so a long or cache-cold conversation can be reset with `/clear` without losing what mattered. Useful once a conversation has run long enough that every turn is re-processing a large, no-longer-cached history — the summary becomes the new starting point instead of the full transcript. Follow the phases in order.

**Plain-language standard.** Applies to the summary this command writes:

@.claude/includes/clarity.md

## Context

- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Worktrees: !`git worktree list 2>/dev/null || echo "(none)"`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Today's date: !`date +%Y-%m-%d`

## Phase 0 — Find the summary's home

1. Current branch matches a ticket branch (`<ticketId>/<slug>/feature`, `<ticketId>/<slug>/quick`, `<ticketId>/<slug>/bug-fix`, or a phase branch `<ticketId>/<phaseSlug>`) → extract `ticketId` and resolve `.specs/<ticketId>_*` (there should be exactly one matching directory) → the summary lives at that directory's `session-summary.md`, alongside the ticket's other docs. Note that for a phase branch, `<phaseSlug>` is the phase's own slug, not the ticket directory's slug — always resolve the directory by glob, never by substituting the branch's slug into the path.
2. Current branch isn't a ticket branch, but this conversation worked on one ticket that has a ticket worktree (as defined in `.claude/includes/worktree-ticket.md`; Context lists the worktrees) → the summary lives in that worktree, at its `.specs/<ticketId>_*/session-summary.md` (absolute path), next to the docs it describes. Don't switch the session there — this command only writes one file.
3. Otherwise → `.claude/session-summaries/<date>-<slug>.md`, where `<slug>` is derived from the conversation's main topic (@.claude/includes/slug.md), creating the directory if needed.
4. That file already exists (an earlier `/codely.compact` in this same thread of work) → read it first. It was the only thing carried into this conversation when it was last cleared, so anything in it still relevant must carry forward into the new summary — this file is the sole memory of everything before its own last write.

## Phase 1 — Write the summary

1. Fill `.claude/templates/session-summary.md` placeholders, drawing on the full conversation available in this context — not just the last few turns.
2. State: the goal of this conversation and where things currently stand, written so someone with zero memory of this conversation understands it on one read.
3. Decisions: every confirmed choice, ruled-out approach, or constraint that would otherwise get silently re-litigated once the context is gone.
4. Key Files: files read, created, or edited that matter for picking the work back up (write "- None" if nothing project-specific was touched — never omit the section).
5. Next Steps: the immediate next action, plus any question still open.
6. Be exhaustive on substance, terse on wording. Nothing outside this file survives the clear the user is about to do — if it's not in the summary, it's gone.
7. Write the file (Phase 0 already decided the mandatory sections above are the whole template — nothing here is optional).

## Phase 2 — Hand off

1. Print the summary in the chat so the user can check it before clearing.
2. Tell the user plainly: this command can't clear the conversation itself — that's a CLI action only they can trigger. Next step is to run `/clear`, then point the fresh conversation at `<path>` (e.g. "continue from `<path>`") to resume with a clean, cheap context.
