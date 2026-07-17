# Bug Fix: {{Title}}

**Branch:** `{{ticketId}}/{{slug}}/bug-fix`
**Ticket ID:** `{{ticketId}}`

## Problem

*Mandatory. The reported symptom and its impact, plus numbered steps to reproduce it. Include environment details (OS/browser/version/deployment) if relevant to reproduction.*

{{problem}}

## Root Cause

*Mandatory. The confirmed defect — code path, why it produces the symptom, and (if known, e.g. via `git blame`/`git log`) when/how it was introduced. Confirmed with the user before the Fix Plan below was drafted.*

{{root_cause}}

## Expected Behavior

*Mandatory. What correct behavior looks like once fixed — concrete enough to become the regression test's assertion.*

{{expected_behavior}}

## Fix Plan

*Mandatory. The chosen approach to address the root cause — not just the symptom — and why, over any alternatives considered.*

{{fix_plan}}

## Out of Scope

*Optional — omit if empty. Related defects noticed during investigation but not fixed here (captured in `follow-up.md` instead).*

{{out_of_scope}}

## Tasks

*Mandatory. Task 1 is always the regression test, written and confirmed failing before any fix code is touched.*

- [ ] Add a regression test reproducing the bug (`{{test_ref}}`) — done when it fails for the root-cause reason, committed on its own
- [ ] {{task}} — done when {{criteria}}

## Key Files

*Mandatory.*

- {{file}}

## Verification

*Mandatory. How the fix is proven, not just asserted.*

### Regression Test

*The record that the bug is fixed: the same test, failing before the fix and passing after, each tied to its own commit. Red is referenced by commit SHA (already committed by the time this is filled in). Green is referenced by commit message instead — that commit doesn't exist yet at the point this section is first written, so look it up via `git log --grep`. If no automated test is feasible for this bug, replace this subsection with a stated reason and rely on Manual Verification below instead.*

- **Test:** `{{test_file}}::{{test_name}}`
- **Before fix (red)** — commit `{{red_commit_sha}}`: `{{command}}`
  ```
  {{failing_output}}
  ```
- **After fix (green)** — commit `"{{fix_commit_message}}"`: `{{command}}`
  ```
  {{passing_output}}
  ```

### Manual Verification

*Optional — omit if the Regression Test above fully covers it. Original repro steps from Problem, re-run against the fix.*

- [ ] {{repro_step}} — no longer reproduces the bug

### Regression Check

- [ ] Full test suite / build / lint run clean
