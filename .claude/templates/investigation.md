# Investigation: {{Title}}

**Ticket ID:** `{{ticketId}}`
**Verdict:** {{Proceed | Proceed with conditions | Do not proceed | Needs more investigation}}

## Question

{{The decision this investigation exists to inform, and who acts on it.}}

## Recommendation

{{Two to four sentences, up front: what was found and what to do about it.}}

## Scope of Investigation

{{What was examined and how — codebase areas, external sources, any prototype.}}

## Out of Scope

{{Adjacent questions this investigation deliberately did not answer.}}

## Key Findings

- {{finding}} — evidence: `{{file:line}}` / {{URL}} / {{prototype result}}

## Options Considered

### Option A: {{name}}

- **How it works:** {{...}}
- **Effort:** {{S | M | L}} — {{rationale}}
- **Risks:** {{...}}
- **Impact on existing code:** {{...}}
- **Reversibility:** {{easy | hard}} — {{why}}

### Option B: {{name}}

- **How it works:** {{...}}
- **Effort:** {{S | M | L}} — {{rationale}}
- **Risks:** {{...}}
- **Impact on existing code:** {{...}}
- **Reversibility:** {{easy | hard}} — {{why}}

## Evaluation

| Criterion                     | Option A | Option B | Option C |
| ----------------------------- | -------- | -------- | -------- |
| Feasibility                   | {{...}}  | {{...}}  | {{...}}  |
| Effort                        | {{...}}  | {{...}}  | {{...}}  |
| Risk                          | {{...}}  | {{...}}  | {{...}}  |
| Maintainability               | {{...}}  | {{...}}  | {{...}}  |
| Alignment with {{constraint}} | {{...}}  | {{...}}  | {{...}}  |

## Risks & Open Unknowns

- {{risk or unknown}} — {{likelihood / impact}} — {{mitigation | accept | resolve before starting}}

## Recommendation Detail

{{The chosen option and why. Conditions that must hold for it to work. What new information would flip the recommendation.}}

## Decision

{{Verdict}} — {{conditions, and follow-ups carried forward}}

## Next Steps

- {{e.g. run /codely.spec for Option A, carrying <condition> into Constraints}}
