# Construction Plan: {{Title}}

**HLD:** `.sdlc/hld.md`
**LLD:** `.sdlc/lld.md`

## Approach

*Mandatory. High-level strategy for turning the HLD's architecture and technology selection into a real, running project structure — how the work is sliced into phases and sequenced.*

{{approach}}

## Out of Scope

*Optional — omit if empty. Infrastructure/tooling explicitly confirmed out of scope for now (e.g. a deployment target deferred to a later pass).*

{{out_of_scope}}

## Decisions

*Optional — omit if empty. "You decide" calls made during planning, with rationale.*

{{decisions}}

## Phases

*Mandatory. Phases run in order — each phase's PR bases on `main`/`master` if the previous phase is already merged, otherwise stacks on the previous phase's branch. Every command a phase will run (scaffolding CLIs, dependency installs, etc.) must be listed in that phase's Commands — execution only ever runs commands already reviewed and approved here, never something improvised later.*

### Phase 1: {{phase_1_title}}

**Branch:** `sys-init/{{phase_1_slug}}`

#### Tasks

- [ ] {{task}} — done when {{criteria}}

#### Commands

*Exact shell commands this phase runs, in order (scaffolding CLIs, dependency installs, etc.). Omit this subsection entirely for a phase that only hand-writes files.*

- `{{command}}`

#### Key Files

- {{file_or_directory}}

<!-- Repeat "### Phase N: {{title}}" with its own Branch/Tasks/Commands/Key Files for each additional phase. -->
