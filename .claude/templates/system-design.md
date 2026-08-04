# System Design: {{Title}}

**Branch:** `{{ticketId}}/{{slug}}/feature`
**Ticket ID:** `{{ticketId}}`

## Architecture

```mermaid
flowchart LR
    ComponentA["{{component_a}}"] -->|"{{interaction}}"| ComponentB["{{component_b}}"]
```

{{architecture_notes}}

## Data Model

```mermaid
erDiagram
    EntityA ||--o{ EntityB : "{{relationship}}"
    EntityA {
        string id
        string field_name
    }
```

## API Contract

| Method | Path | Request | Response | Auth |
|---|---|---|---|---|
| {{method}} | {{path}} | {{request_schema}} | {{response_schema}} | {{auth}} |

## Component Breakdown

- **{{component_name}}** — {{responsibility}}

```mermaid
classDiagram
    class Component {
        +string id
        +type fieldName
        +operationName() ReturnType
    }
```

## Decisions

{{decisions}}
