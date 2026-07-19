# codely.sdd

Coginitics' Spec-Driven Development (SDD) toolkit — a set of Claude Code commands and templates for running client projects through a consistent requirements → design → plan → implement → QA cycle.

## Structure

```
.claude/commands/
├── codely.quick.md         ← /codely.quick — fast path for small, well-understood changes
├── codely.spec.md          ← /codely.spec — turns a requirement into a spec
├── codely.plan.md          ← /codely.plan — turns a spec into an implementation plan
├── codely.implement.md     ← /codely.implement — executes an implementation plan
├── codely.qa.md            ← /codely.qa — verifies an implementation against its spec/plan
├── codely.follow-up.md     ← /codely.follow-up — captures post-delivery feedback and next steps
├── codely.bug-fix.md       ← /codely.bug-fix — reproduce, root-cause, and fix a reported bug behind a red→green regression test
├── codely.project-plan.md  ← /codely.project-plan — project-level plan across multiple specs
├── codely.ui.md            ← /codely.ui — approved spec to previewable screen mockups + design breakdown
├── sdlc.plan.md            ← /sdlc.plan — SDLC planning phase: idea → approved project charter + project plan
└── sdlc.req-analysis.md    ← /sdlc.req-analysis — SDLC requirements analysis phase: charter → approved SRS + use-case model

.claude/templates/
├── spec.md              ← requirements spec template
├── system-design.md     ← system design / API contract / component breakdown template
├── design.md             ← screen inventory / component breakdown template, paired with design/*.html mockups
├── plan.md               ← implementation plan template
├── quick-plan.md         ← combined spec+plan template for /codely.quick
├── bug-fix-plan.md       ← combined investigation+fix template for /codely.bug-fix
├── follow-up.md          ← post-delivery follow-up template
├── project-charter.md    ← SDLC project charter template (goals/scope/feasibility/risk), used by /sdlc.plan
├── project-plan.md       ← SDLC project plan template (roadmap/milestones/resourcing), used by /sdlc.plan
├── use-case-model.md     ← SDLC use-case model template (actors/use cases/user stories/workflows), used by /sdlc.req-analysis
└── srs.md                ← SDLC Software Requirements Specification template, used by /sdlc.req-analysis

.claude/includes/
└── (shared snippets/files referenced by commands and templates — TBD)
```

## Status

This repo is currently scaffolded only — command and template files are stubs (title + TODO). Content will be filled in as each command is implemented.

See [WORKFLOW.md](./WORKFLOW.md) for the intended end-to-end SDD process (to be documented).