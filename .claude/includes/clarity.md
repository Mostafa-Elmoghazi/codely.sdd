<!--
  Shared writing standard — referenced by every /codely.*, /sdlc.*, and /trainer.* command.
  Not per-project config: it applies as-is on every project, with no first-run prompt.
  Read it before asking the user anything and before writing any output file.
-->

# Clarity Standard

Every question this command puts to the user, and every file it writes, must be in
**plain language** — the kind a smart non-specialist on the team could follow without a
glossary. This is a hard requirement, not a stylistic preference. It never overrides a
project's `CLAUDE.md`, but it applies wherever `CLAUDE.md` is silent.

## Before asking the user anything

When a question introduces an idea, concept, option, trade-off, or term the user hasn't
raised themselves:

1. **Explain it first, briefly.** One or two sentences in everyday words: what it is and
   why it matters here. Define any unavoidable technical term in the same sentence it
   first appears.
2. **Then ask.** Keep the question and its answer options concrete and short. No option
   label should need a further explanation to be understood.
3. **Don't stack jargon.** If a single question needs more than one new term explained,
   split it into smaller questions.
4. **Match the user's level.** Mirror the vocabulary the user has been using. Don't
   introduce framework names, patterns, or acronyms just to sound precise.

## In every file this command writes

- **Lead with the plain-language version.** Before any detailed or formal treatment of
  an idea, state in one or two sentences what it is and why it's here. A reader who
  stops after the first paragraph of a section should still come away with the gist.
- **Spell out terms on first use.** Expand each acronym once; gloss a domain or
  technical term in a half-sentence the first time it appears in the document.
- **Be brief.** Short sentences, one idea each. Cut hedging, filler, and restatement.
  Prefer a concrete example over an abstract description.
- **No decoration.** No throat-clearing, no piling up synonyms, no "as we all know."
  Say it once, clearly, and move on.
- **Keep the structure scannable.** Headings, short lists, and small tables so a reader
  can find what they need without reading top to bottom.

## The test

If a teammate who isn't a specialist in this area couldn't read the question (or the
section) once and explain back what it means and what's being decided, it isn't plain
enough yet — rewrite it.
