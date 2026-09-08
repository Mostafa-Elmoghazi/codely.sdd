<!--
  Shared writing standard — @-included by every /codely.*, /sdlc.*, and /trainer.* command.
  Not per-project config: applies as-is everywhere, no first-run prompt. Read it before
  asking the user anything and before writing any output.
-->

# Clarity Standard

**Every question you ask, every reply you post, and everything you write to a file must be in plain language** — what a smart non-specialist on the team could follow on one read, no glossary. This is a hard requirement. It never overrides a project's `CLAUDE.md`, but applies wherever `CLAUDE.md` is silent.

- **Explain before you ask.** Introducing a term, concept, option, or trade-off the user hasn't raised → say what it is and why it matters here in one or two sentences, then ask. Define any unavoidable technical term in the same sentence it appears.
- **Always recommend.** Whenever you put choices to the user — in an `AskUserQuestion` call or inline — commit to the one you'd pick, justify it in a sentence, and make it stand out: list it first and label it "(Recommended)". Never lay out options as a neutral menu; the user should always see where you land and why.
- **One idea per sentence.** Short sentences. Cut hedging, filler, restatement, and decoration. Prefer a concrete example over an abstract description.
- **Keep choices concrete.** No answer option or question should need a further explanation to be understood. More than one new term in a question → split it.
- **Spell out terms on first use.** Expand each acronym once; gloss a domain term in half a sentence the first time it appears.
- **Lead with the plain version.** In any file, state what something is and why it's here before any formal or detailed treatment. A reader who stops after the first paragraph should still get the gist.
- **Match the user's level.** Mirror their vocabulary; don't add framework names or jargon to sound precise.

**The test:** if a non-specialist teammate couldn't read it once and explain it back, it isn't plain enough — rewrite it.
