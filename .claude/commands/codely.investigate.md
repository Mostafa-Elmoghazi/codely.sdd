---
description: Investigate and evaluate a proposed piece of work before committing to it — produce a recommendation, not code or a plan
argument-hint: [ticket-id] [question or proposed work]
allowed-tools: Read, Glob, Grep, Write, Edit, AskUserQuestion, WebFetch, WebSearch, Bash
disable-model-invocation: true
---

# codely.investigate

> **Scope guardrail:** never writes or modifies application code, and never touches files outside `.claude/includes/ticket-format.md` and `.specs/<ticketId>_<slug>/` (only `investigation.md` and `summary.md`, plus `follow-up.md` if deferred items surface). Any prototype or experiment runs entirely inside the session scratchpad and is deleted before the doc is written — only its findings survive, as evidence. Codebase and web exploration are read-only. One chat-only gate — the decision — stands before any git-mutating step. `ticket-format.md`, if drafted, is committed immediately as shared project config, independent of the decision.

Turn a ticket ID + a question about a proposed piece of work into an evidence-backed **investigation** under `.specs/`, ending in one of four verdicts: **Proceed**, **Proceed with conditions**, **Do not proceed**, **Needs more investigation**. This runs _before_ `/codely.spec` whenever there's real uncertainty about whether or how the work should be done; a Proceed verdict feeds the spec. It produces a decision-support document only — no spec, no plan, no code. Follow the phases in order. No git changes before the decision, except the shared-config files noted above. If a `CLAUDE.md` exists, it overrides this file wherever they conflict.

**Plain-language standard.** Applies to every question this command asks and every file it writes:

@.claude/includes/clarity.md

**Document authoring.** Applies to every file this command writes from a template:

@.claude/includes/authoring.md

## Context

- Working tree status: !`git status --porcelain 2>/dev/null || echo "(not a git repository, or git unavailable)"`
- Current branch: !`git branch --show-current 2>/dev/null || echo "(none — repository may have zero commits)"`
- Repository has commits: !`git rev-parse --verify HEAD >/dev/null 2>&1 && echo yes || echo no`
- `.specs/` contents: !`ls -1 .specs 2>/dev/null || echo "(.specs/ does not exist yet)"`
- Ticket-format config present: !`test -f .claude/includes/ticket-format.md && echo yes || echo no`

## Phase 0 — Resolve inputs

1. Read `CLAUDE.md` if present.
2. `$ARGUMENTS` non-empty: first token (minus trailing `:`) = ticket ID, rest = the question / proposed work.
3. `$ARGUMENTS` empty, or missing the ticket ID and/or the question: ask the user for whatever's missing. Don't continue without both.

## Phase 1 — Load the ticket ID format

1. Read `.claude/includes/ticket-format.md`.
2. **Missing** (first run): tell the user, then AskUserQuestion — "describe the format, I'll draft the file" vs. "I'll set it up myself, stop here." If they describe it, ask the scheme (JIRA/GitHub/Linear/none/other), write the file, then stage and commit it on whatever branch is currently checked out: `Add ticket ID format config`. Otherwise stop.
3. **Present:** extract the pattern/regex and examples.
4. Validate the candidate ticket ID against it.
5. **No match** (or no candidate): show the expected format, AskUserQuestion — corrected ID / use anyway / no-ticket fallback. Never guess.

## Phase 2 — Check for an existing investigation

1. Glob `.specs/` for `<ticketId>_*`.
2. **None:** go to Phase 3.
3. **Found with `investigation.md`:** read + summarize it, AskUserQuestion — Resume/revise (reuse slug, keep prior findings, extend) / Start over (reuse slug, re-investigate) / Cancel. Either way skip Phase 3 and reuse the existing slug.
4. **Found with `spec.md` or `plan.md` but no `investigation.md`:** the work is already past the investigation stage — tell the user, AskUserQuestion — investigate anyway (a genuinely new open question about the same ticket) / cancel and point them at the existing spec/plan.

## Phase 3 — Generate the slug (new tickets only)

@.claude/includes/slug.md

## Phase 4 — Frame the investigation

**Mandatory — don't investigate until the frame is confirmed.** An unframed spike sprawls.

1. Restate the question as a **decision to be made** — what choice does this investigation exist to inform, and who acts on it.
2. List the **specific unknowns** that block that decision, as concrete questions each with a way to answer it (read code X, test library Y, prototype Z, check constraint W).
3. State what **"enough evidence"** looks like — the bar at which a verdict can be given without over-investigating.
4. State what's **out of scope** for this investigation — adjacent questions it deliberately won't answer.
5. AskUserQuestion to confirm the frame: the decision, the unknowns list, the out-of-scope calls. Adjust until the user agrees, then proceed.

## Phase 5 — Investigate

Work through the unknowns from Phase 4, gathering evidence for each. **Start with the codebase** — most of the answer is usually already there.

### Codebase (read-only)

Glob/Grep/Read across the repo for anything that bears on the decision:

- **Prior art** — similar or overlapping features already built; how they solved the same class of problem.
- **Reusable building blocks** — existing utilities, abstractions, hooks, components, schemas, services the proposed work could extend instead of reinventing.
- **Conventions & constraints** — patterns the codebase already commits to (state management, data access, error handling, styling, i18n, testing) that any approach here must fit.
- **Related specs** — other `.specs/<ticketId>_*/` directories touching the same area; decisions already recorded there.
- **Existing tests** — coverage around the affected area that shows intended behavior and edge cases, and that a change would have to keep green.
- **History** — `git log`/`git blame` on the affected files for past attempts, reverts, or context on why the current shape exists.
- **Dependencies already present** — `package.json` / lockfile / equivalent: is a library that would help already a project dependency, and at what version.

Cite every finding as `file:line` (or commit SHA).

### External (read-only)

Only for unknowns the codebase can't answer: WebSearch / WebFetch for library capabilities and limits, version compatibility, known issues, standards, prior art outside the repo. Cite the URL.

### Prototype (optional, throwaway)

If an unknown can only be settled by running something, build the smallest possible experiment **in the session scratchpad only** — never in the repo tree. Capture the result as a finding, then delete the experiment. No prototype code reaches `.specs/` or the repo.

## Phase 6 — Enumerate options

Identify 2–4 viable approaches to the proposed work. **Always include a "defer / do nothing" option** as the baseline. For each option record: how it works, effort (S/M/L with rationale), risks, blast radius on existing code, reversibility (how hard to undo if it's wrong).

## Phase 7 — Evaluate

Score every option against a fixed set of criteria — at minimum: feasibility, effort, risk, maintainability, and alignment with the project's constraints (from `CLAUDE.md`, existing conventions, and any `.sdlc/` docs). Call out any **show-stopper** explicitly rather than folding it into a score.

## Phase 8 — Interview the user (rounds)

Only for what evidence alone can't settle: risk tolerance, priority, an acceptable effort ceiling, which criteria matter most, constraints not written down anywhere.

@.claude/includes/interview.md

Record any "you decide" call. Points that matter but sit outside this decision → tell the user they'll go to a follow-up file, and hold them for Phase 9.

## Phase 9 — Deferred items

1. Phase 5 or 8 surfaced points worth keeping but outside this decision → tell the user they'll be captured for later triage.
2. Create (or append to) `.specs/<ticketId>_<slug>/follow-up.md` from `.claude/templates/follow-up.md` with those items.
3. These get triaged one at a time later via `/codely.follow-up`.

## Phase 10 — Write the investigation

1. Fill `.claude/templates/investigation.md` placeholders.
2. Mandatory: Question (the decision this informs), Recommendation (2–4 sentences, up front), Scope of Investigation, Key Findings (each with `file:line` or URL evidence), Options Considered (the per-option block, repeated), Evaluation (the criteria table), Risks & Open Unknowns, Recommendation Detail (chosen option, why, conditions, what would flip it).
3. Set **Verdict** to one of: `Proceed` / `Proceed with conditions` / `Do not proceed` / `Needs more investigation`. Leave the Decision section as a placeholder — Phase 11 fills it.
4. Optional (omit a section entirely when empty): Out of Scope, Next Steps.
5. Write to `.specs/<ticketId>_<slug>/investigation.md`, creating the directory if needed.
6. **Write the plain-language summary** to `.specs/<ticketId>_<slug>/summary.md` from `.claude/templates/investigation-summary.md`. This is for a reader who has not seen the code or the full investigation — a product owner, a new teammate, a stakeholder. Rules: everyday words; each finding and decision leads with one bold sentence and then explains the reasoning; explain any technical term in the same sentence it first appears; no `file:line` citations or URLs (those live in `investigation.md`); name the ideas the reader would expect and say why they were set aside; keep it to roughly one page. It must agree with `investigation.md` on every fact and verdict — it is a translation, not a second opinion.
7. Confirm the scratchpad prototype/experiment, if any, has been deleted.
8. Remind the user: no application code has changed — only the investigation, summary (and possibly follow-up) files.

## Phase 11 — Decision loop

1. Present the recommendation and ask the user to pick the verdict: **Proceed** / **Proceed with conditions** / **Do not proceed** / **Needs more investigation**. Loop (revising the doc) until they pick one.
2. Write the chosen verdict into **Verdict**, and into the **Decision** section: the verdict, any conditions, and any follow-ups carried forward. The decision is a chat-only gate — never add an approver name to the file.
3. Update `summary.md` to match: its **Verdict** line, and its "What we decided" / "What's carried forward" sections, so the two files never disagree.

## Phase 12 — Preserve the record & next steps

**Don't start until Phase 11's verdict is explicit.**

1. `git status --porcelain`, disregarding `investigation.md` / `summary.md` / `follow-up.md` just written — anything else dirty → stop, AskUserQuestion (commit / stash / discard / abort). Never auto-act.
2. Trunk: `CLAUDE.md`'s choice, else `main`, else `master`. Ask if still ambiguous.
3. Zero commits (`git rev-parse --verify HEAD` fails): tell the user, skip switch/pull, ask whether to branch from the unborn state or commit first.
4. Branch by verdict — never commit `investigation.md` straight to trunk:
   - **Proceed** / **Proceed with conditions:** switch to trunk if needed (pull only if it tracks a remote), then create/checkout `<ticketId>/<slug>/feature` — the ticket's docs branch, so `/codely.spec` continues on it. If it already exists, check it out and note how far ahead of trunk it is.
   - **Do not proceed** / **Needs more investigation:** AskUserQuestion — preserve on a `<ticketId>/<slug>/investigation` branch (create/checkout it) / leave `investigation.md` and `summary.md` uncommitted for the user to handle / discard them.
5. On a preserve path: stage only `investigation.md` and `summary.md` (and `follow-up.md` if created), commit `Add investigation for <ticketId>`. Never push.
6. Confirm: investigation path, summary path, verdict, current branch. Then:
   - **Proceed:** run `/clear`, then `/codely.spec` — the investigation's recommended option and findings are the spec's starting point.
   - **Proceed with conditions:** same, and tell the user to carry the conditions into the spec's Constraints & Assumptions.
   - **Do not proceed:** done — nothing further unless the question changes.
   - **Needs more investigation:** state the narrowed unknowns for the next `/codely.investigate` run.
