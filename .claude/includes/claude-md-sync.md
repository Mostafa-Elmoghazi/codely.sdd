<!--
  Shared CLAUDE.md reconciliation rule.
  @-included as a step near the end of implementation by every command that changes
  application code (codely.implement, codely.quick, codely.bug-fix). codely.sys-init
  gets its own variant in-line, since it also creates brand-new apps/packages.
-->

For every app/package this phase's changes touched: check for a `CLAUDE.md` at its root (e.g. `packages/<name>/CLAUDE.md`, `apps/<name>/CLAUDE.md`). Missing → skip it; creating one isn't this command's job. Present → update it only if something it documents is now wrong or incomplete — a new file/module it should list, a changed convention or invariant, something it says that's now obsolete. Nothing changed at that level → leave it alone. Stage any update in the same commit as the rest of this phase's changes, not a follow-up.
