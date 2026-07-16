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
├── codely.bug-fix.md       ← /codely.bug-fix — structured bug reproduction and fix workflow
├── codely.project-plan.md  ← /codely.project-plan — project-level plan across multiple specs
└── codely.ui.md            ← /codely.ui — UI/UX requirement to component breakdown

.claude/templates/
├── spec.md            ← requirements spec template
├── system-design.md   ← system design / API contract / component breakdown template
├── plan.md            ← implementation plan template
├── quick-plan.md       ← combined spec+plan template for /codely.quick
└── follow-up.md        ← post-delivery follow-up template

.claude/includes/
└── (shared snippets/files referenced by commands and templates — TBD)
```

## Status

This repo is currently scaffolded only — command and template files are stubs (title + TODO). Content will be filled in as each command is implemented.

See [WORKFLOW.md](./WORKFLOW.md) for the intended end-to-end SDD process (to be documented).