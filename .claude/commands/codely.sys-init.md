---
description: Turn the approved system design into a real project scaffold, phase by phase, via an interviewed construction plan
argument-hint: [phase-number]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, Bash
disable-model-invocation: true
---

# codely.sys-init

> **Scope guardrail:** the only command in this family allowed to write, modify, or install real project files/dependencies outside `.sdlc/`/`.specs/` — and only by executing a construction plan the user already interviewed and approved. Never runs a scaffolding/install command that isn't already listed in that phase's approved Commands. Outputs are `.sdlc/construction-plan.md` (committed to its own `sys-init/plan` branch) and, per phase thereafter, real project files/directories plus a commit + push + PR on a `sys-init/<phase-slug>` branch.

Turn the approved `.sdlc/hld.md`/`.sdlc/lld.md` into a real project structure, one reviewed phase at a time. The first run drafts and gets approval for `.sdlc/construction-plan.md` — a phased breakdown of the scaffolding work, including the exact commands each phase will run. Every later run executes the next unchecked phase: branch, scaffold, verify, approve, commit, push, PR. Re-run this command (after `/clear`) once each phase's PR is merged, until every phase is checked off. Follow the phases in order. No git changes before an approval. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.sdlc/` contents: !`ls -1 .sdlc 2>/dev/null || echo "(.sdlc/ does not exist yet)"`
- `gh` CLI available: !`gh --version >/dev/null 2>&1 && echo yes || echo no`

## Phase 0 — Preconditions and mode

1. Read `CLAUDE.md` if present.
2. Glob `.sdlc/hld.md` and `.sdlc/lld.md`. **Missing either:** tell the user project scaffolding needs an approved system design first, stop, recommend `/sdlc.system-design`.
3. Trunk: `CLAUDE.md`'s choice, else `main`, else `master`.
4. Glob `.sdlc/construction-plan.md`:
   - **Missing:** go to Phase 1 (**Planning mode**).
   - **Present:** read it, parse every phase's tasks.
     - **Every phase fully checked:** tell the user project scaffolding is complete, stop, recommend `/codely.backlog` (if `.sdlc/backlog.md` doesn't exist yet) and `/codely.spec <ticketId> <description>` per ticket.
     - **No phase has any checked task yet** (approved, but nothing executed): summarize the plan, AskUserQuestion — proceed to execute Phase 1 as planned / revise the plan first (return to Phase 2) / cancel. "Revise" is only offered here, before any real scaffolding exists to conflict with.
     - **At least one phase has progress:** go to Phase 4 (**Execution mode**). Target phase = `$ARGUMENTS`' phase number if given (validate it exists), else the phase matching current branch `sys-init/<phaseSlug>` if on one, else the first phase with any unchecked task.

## Phase 1 — Load context (planning mode)

1. Read `.sdlc/hld.md` and `.sdlc/lld.md` — Architecture Overview, Major Modules, Technology Selection, Component Design bound the scaffold.
2. Glob `.sdlc/project-plan.md` — present → read it too, its roadmap can inform phase sequencing.
3. Glob `.sdlc/ui-template/tokens.css` and `.sdlc/ui-template.md` — present → read them too; a UI-facing phase can then offer seeding a real theme/tokens file (Tailwind config, CSS custom properties, etc.) from these chosen tokens instead of leaving that to whatever the scaffolding CLI defaults to.
4. Glob/Grep/Read the repo root — don't propose scaffolding what already exists; note what's already in place so later phases can skip or extend it instead of colliding.

## Phase 2 — Derive candidate phases (planning mode)

1. From the HLD's Architecture Overview/Major Modules/Technology Selection and the LLD's Component Design, derive a dependency-ordered phase breakdown. Typical shape (adapt freely to the actual stack — merge, split, drop, or reorder as it warrants): repo/tooling baseline (package manager, lint/format config, git hooks) → one phase per major module/service → data layer/migrations → CI → deployment pipeline. If `.sdlc/ui-template/tokens.css` exists and the stack has a UI-facing phase, fold in a task seeding that phase's real theme/tokens config from it.
2. For each candidate phase work out: title, branch slug, ordered tasks with done-criteria, the exact scaffolding-CLI/install/other shell commands it will run (if any — a phase may be hand-written files only), and key files/directories touched.

## Phase 3 — Interview, write, and approve the plan (planning mode)

**Mandatory — don't write the plan until the phase breakdown, sequencing, and every phase's exact commands are confirmed.**

1. AskUserQuestion in rounds of up to 4-5 (cap is 4/call) — confirm the phase breakdown and, for each phase that runs commands, the exact commands themselves. This is the review gate for anything Execution mode will later run — nothing improvised gets in.
2. Digest each round; run another if new questions surface.
3. On "you decide," state a recommendation and get explicit confirmation — record it under Decisions.
4. Fill `.claude/templates/construction-plan.md` placeholders. Mandatory: Approach (high-level strategy for turning the HLD's architecture and technology selection into a real, running project structure — how the work is sliced into phases and sequenced), Phases (each with Branch/Tasks/Key Files, and Commands unless the phase only hand-writes files — repeat `### Phase N: {{title}}` for each additional phase; phases run in order, each phase's PR bases on `main`/`master` if the previous phase is already merged, otherwise stacks on the previous phase's branch). Optional (omit a section entirely when empty): Out of Scope (infrastructure/tooling explicitly confirmed out of scope for now, e.g. a deployment target deferred to a later pass), Decisions. Write to `.sdlc/construction-plan.md`.
5. Ask the user to review it — approve or request changes, loop until approved.
6. Approved → `git status --porcelain`, disregarding the newly-written `construction-plan.md` — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort).
7. Zero commits (`git rev-parse --verify HEAD` fails): tell the user, skip switch/pull, ask whether to branch from the unborn state or commit first. Otherwise switch to trunk if needed, pull only if it tracks a remote.
8. Create/checkout `sys-init/plan`. Already exists → check it out, note commits ahead of trunk.
9. Stage and commit `construction-plan.md`: `Add project construction plan`.
10. Push, setting upstream if none. `gh` available → `gh pr create --draft` (base = trunk, head = `sys-init/plan`, body noting this is a docs-only plan, no scaffolding yet). `gh` unavailable → tell the user to push and open the PR manually.
11. Confirm the plan path and PR (or manual-PR instructions). Tell the user: get this PR reviewed/merged, then run `/clear` then `/codely.sys-init` again to execute Phase 1.

## Phase 4 — Prepare the phase branch (execution mode)

1. `git status --porcelain`. Dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Base = trunk if this is the first phase or the previous phase's branch is already merged into trunk; otherwise the previous phase's branch. Check "merged" via `git branch --merged <trunk>` or, if `gh` is available, the previous phase's PR state.
3. Fetch/pull the base if it tracks a remote.
4. Create/checkout `sys-init/<phaseSlug>` from that base. Already exists → check it out, note tasks already checked (resuming).

## Phase 5 — Execute the phase's tasks

1. Work through the phase's unchecked tasks in order.
2. A task's target file/directory already exists and is non-empty → skip it, note it as already present rather than overwriting (supports resuming a scaffold the user partly did by hand, or a previous partial run). Empty or missing → proceed normally.
3. Run only the commands already listed under that phase's Commands in `construction-plan.md`. A task turns out to need a command that isn't listed → don't run it silently: AskUserQuestion whether to add it to the plan (then update `construction-plan.md`'s Commands list) or handle it another way.
4. Follow the conventions already established by any modules earlier phases scaffolded.
5. Mark a task `- [x]` in `construction-plan.md` as soon as its done-criteria is met — keeps progress resumable if the run stops early.
6. Necessary work surfaces outside this phase's task list → don't do it silently. Flag it to the user; capture it as a new phase or under Decisions if they want it kept.

## Phase 6 — Verify

1. Detect what's reasonable to verify for what this phase just scaffolded: `CLAUDE.md` first, then `package.json` scripts / `Makefile` / equivalent for anything newly scaffolded (does it install cleanly, build, lint, boot). Still unclear → tell the user this needs manual verification rather than guessing at commands that might not exist.
2. Run them. Failures → fix and re-run.

## Phase 7 — Approval loop

1. Summarize the phase's changes — files/directories created, commands run, verification results.
2. Approve or request changes, loop until approved.
3. Approval is a chat-only gate for Phase 8 — never add an approver name to any file.

## Phase 8 — Commit, push, PR

**Don't start until Phase 7 approval is explicit.**

1. Stage the new/changed project files plus `construction-plan.md` (checkbox updates).
2. Commit: `Scaffold <phase title> for project init`.
3. Push the phase branch, setting upstream if it has none.
4. `gh` available → `gh pr create --draft` (base = the branch from Phase 4, head = the phase branch, title referencing the phase, body linking back to `construction-plan.md`'s phase section). `gh` unavailable → tell the user to push and open the PR manually.
5. Confirm: files/directories changed, commit, PR URL (or manual-PR instructions).

## Phase 9 — Next steps

1. `construction-plan.md` still has unchecked tasks in other phases → tell the user to get this PR reviewed/merged, then run `/clear` then `/codely.sys-init` again for the next phase.
2. Every phase now fully checked → tell the user project scaffolding is complete; recommend `/codely.backlog` (if `.sdlc/backlog.md` doesn't exist yet) and `/codely.spec <ticketId> <description>` per ticket to start implementation.
