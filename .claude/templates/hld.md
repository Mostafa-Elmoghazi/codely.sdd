# High-Level Design: {{Title}}

**Charter:** `.sdlc/project-charter.md`
**SRS:** `.sdlc/srs.md`
**Use-Case Model:** `.sdlc/use-case-model.md`

## Status

- [ ] Architecture Overview
- [ ] Data Model
- [ ] Technology Selection
- [ ] UI/UX Overview

## Architecture Overview

```mermaid
flowchart LR
    subgraph Client["{{client_layer}}"]
        ClientApp["{{client_component}}"]
    end
    subgraph Server["{{server_layer}}"]
        ServerApp["{{server_component}}"]
    end
    ClientApp -->|"{{protocol}}"| ServerApp
```

## Major Modules

- **{{module_name}}** — {{responsibility}}

## Data Model

```mermaid
erDiagram
    EntityA ||--o{ EntityB : "{{relationship}}"
    EntityA {
        string id
        string field_name
    }
    EntityB {
        string id
    }
```

## Technology Selection

- **{{category}}** ({{e.g. Language, Framework, Database, Hosting}}): {{choice}} — {{rationale}}

## UI/UX Overview

- **Key Screens:** {{screen_list}}

```mermaid
flowchart TD
    ScreenA["{{screen_a}}"] --> ScreenB["{{screen_b}}"]
```

## Traceability

| Decision | Requirement / Use Case |
|---|---|
| {{decision}} | {{fr_or_uc_reference}} |

## Decisions

{{decisions}}
