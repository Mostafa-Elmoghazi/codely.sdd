# codely.sdd

Coginitics' Spec-Driven Development (SDD) toolkit — a set of Claude Code commands and templates for running client projects through a consistent requirements → design → plan → implement → QA cycle.

## Structure

```
.claude/commands/
├── codely.quick.md         ← /codely.quick — fast path for small, well-understood changes
├── codely.backlog.md       ← /codely.backlog — approved system design to a proposed, board-agnostic ticket backlog
├── codely.spec.md          ← /codely.spec — turns a requirement into a spec
├── codely.sys-design.md    ← /codely.sys-design — optional: approved spec to ticket-level system design (architecture/data/API/components)
├── codely.plan.md          ← /codely.plan — turns a spec into an implementation plan
├── codely.implement.md     ← /codely.implement — executes an implementation plan
├── codely.qa.md            ← /codely.qa — verifies an implementation against its spec/plan
├── codely.follow-up.md     ← /codely.follow-up — captures post-delivery feedback and next steps
├── codely.bug-fix.md       ← /codely.bug-fix — reproduce, root-cause, and fix a reported bug behind a red→green regression test
├── codely.ui.md            ← /codely.ui — approved spec to previewable screen mockups + design breakdown
├── sdlc.plan.md            ← /sdlc.plan — SDLC planning phase: idea → approved project charter + project plan
├── sdlc.req-analysis.md    ← /sdlc.req-analysis — SDLC requirements analysis phase: charter → approved SRS + use-case model
└── sdlc.system-design.md   ← /sdlc.system-design — SDLC system design phase: SRS/use-case model → approved HLD + LLD

.claude/templates/
├── spec.md              ← requirements spec template
├── system-design.md     ← ticket-level system design template (architecture/data model/API contract/components), used by /codely.sys-design
├── design.md             ← screen inventory / component breakdown template, paired with design/*.html mockups
├── plan.md               ← implementation plan template
├── quick-plan.md         ← combined spec+plan template for /codely.quick
├── bug-fix-plan.md       ← combined investigation+fix template for /codely.bug-fix
├── follow-up.md          ← post-delivery follow-up template
├── project-charter.md    ← SDLC project charter template (goals/scope/feasibility/risk), used by /sdlc.plan
├── project-plan.md       ← SDLC project plan template (roadmap/milestones/resourcing), used by /sdlc.plan
├── use-case-model.md     ← SDLC use-case model template (actors/use cases/user stories/workflows), used by /sdlc.req-analysis
├── srs.md                ← SDLC Software Requirements Specification template, used by /sdlc.req-analysis
├── hld.md                ← SDLC High-Level Design template (architecture/data model/tech stack), used by /sdlc.system-design
├── lld.md                ← SDLC Low-Level Design template (component/class design/API contracts), used by /sdlc.system-design
└── backlog.md            ← proposed ticket backlog template (waves/dependencies/sizing), used by /codely.backlog

.claude/includes/
└── (shared snippets/files referenced by commands and templates — TBD)
```

## Pipeline

```
sdlc.plan → sdlc.req-analysis → sdlc.system-design → codely.backlog
  → [per ticket] codely.spec → codely.sys-design (optional) → codely.ui (optional)
    → codely.plan → codely.implement → codely.qa → codely.follow-up
```

`sdlc.*` commands are chat-only (no git actions) and write project-wide artifacts under `.sdlc/`. `codely.*` commands work ticket-by-ticket under `.specs/<ticketId>_<slug>/` and drive the actual branch/commit/PR flow. `codely.quick` and `codely.bug-fix` are alternate entry points for small changes and bug fixes respectively, bypassing most of the above.

## Status

All commands and templates listed above are implemented. See [WORKFLOW.md](./WORKFLOW.md) for a fuller narrative walkthrough (to be documented).