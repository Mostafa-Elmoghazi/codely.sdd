# Course: {{Title}}

**Slug:** `{{slug}}`
**Branch:** `course/{{slug}}`

## Overview

*Mandatory. What the course covers and what the learner can do by the end.*

{{overview}}

## Level & Audience

**Level:** {{level}}
**Audience:** {{audience}}

## Prerequisites

*Optional — omit if none.*

{{prerequisites}}

## Decisions

*Optional — omit if empty. "You decide" calls made during the interview, with rationale.*

{{decisions}}

## Curriculum

*Mandatory. Modules lettered A, B, C…; lessons numbered 1, 2, 3… within each module — a lesson's id is `{{module}}.{{lesson}}` (e.g. `A.1`). Every module's final lesson is a `(Workshop)` — hands-on practice or a small project for that module. `/trainer.train` ticks a lesson `- [x]` once taken, appending `(Quiz: {{score}})` or `(Project: done — {{summary}})` where applicable, and ticks a module's own checkbox only once every lesson under it is ticked. Nothing below this point is edited by `/trainer.start` once approved — only `/trainer.train` writes here after that.*

- [ ] **Module A — {{module_a_title}}**
  - [ ] A.1 {{lesson_title}}
  - [ ] A.2 {{lesson_title}}
  - [ ] A.3 (Workshop) {{workshop_title}}

<!-- Repeat "- [ ] **Module X — {{title}}**" (B, C, D…) with its own lettered lessons, always ending in a "(Workshop)" lesson, for each additional module. -->