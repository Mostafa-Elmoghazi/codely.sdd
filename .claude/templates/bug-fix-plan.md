# Bug Fix: {{Title}}

**Branch:** `{{ticketId}}/{{slug}}/bug-fix`
**Ticket ID:** `{{ticketId}}`

## Problem

{{problem}}

## Root Cause

{{root_cause}}

## Expected Behavior

{{expected_behavior}}

## Fix Plan

{{fix_plan}}

## Out of Scope

{{out_of_scope}}

## Tasks

- [ ] Add a regression test reproducing the bug (`{{test_ref}}`) — done when it fails for the root-cause reason, committed on its own
- [ ] {{task}} — done when {{criteria}}

## Key Files

- {{file}}

## Verification

### Regression Test

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

- [ ] {{repro_step}} — no longer reproduces the bug

### Regression Check

- [ ] Full test suite / build / lint run clean
