<!--
  Shared document-authoring rules.
  @-included near the top of every command that fills a .claude/templates/ template and
  writes the result under .specs/ / .sdlc/ / .courses/. Governs structure and the
  approval record; clarity.md (its sibling) governs plain language.
  Each phase still names which sections are mandatory vs optional for that document —
  this file only states the rules that were repeated verbatim across every command.
-->

# Document Authoring Rules

For every file this command writes from a `.claude/templates/` template:

- **The template is headers + placeholder tokens only.** Use each phase's guidance for what belongs in a section; never carry that guidance into the finished file.
- **Fill every mandatory section**, even if briefly.
- **Omit an optional section entirely when it has no content.** Never leave a heading with "N/A", "None", or "TBD" under it.
- **The approval gate is chat-only.** Never write an approver name, sign-off, or date into the document — approval lives in the conversation, not the file.
