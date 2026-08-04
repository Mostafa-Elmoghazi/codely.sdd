# Use-Case Model: {{Title}}

**Charter:** `.sdlc/project-charter.md`

## Status

- [ ] Actors
- [ ] Use Cases
- [ ] User Stories
- [ ] Process Workflows

## Actors

- **{{actor_name}}** — {{actor_description}}

## Use Cases

```mermaid
flowchart LR
    subgraph System["{{system_name}}"]
        UC1(["{{use_case_title}}"])
        UC2(["{{use_case_title}}"])
    end
    Actor1["{{actor_name}}"] --> UC1
    Actor1 --> UC2
```

### {{use_case_title}}

**Actor(s):** {{actor_name}}
**Goal:** {{goal}}
**Preconditions:** {{preconditions}}
**Main Flow:** {{main_flow}}
**Alternate/Exception Flows:** {{alternate_flows}}

## User Stories

- As a {{role}}, I want {{capability}}, so that {{benefit}}.

## Process Workflows

### {{workflow_title}}

```mermaid
sequenceDiagram
    participant {{actor_1}}
    participant System
    {{actor_1}}->>System: {{step}}
    System-->>{{actor_1}}: {{step}}
```

## Notes & Decisions

{{decisions}}
