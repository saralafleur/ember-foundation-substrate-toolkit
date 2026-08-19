---
name: document-assembly
argument-hint: "assemble <path to validated findings> --target <output-dir>"
description: >
  Turns validated, fact-checked findings (from a corpus-mining pass, or any
  equivalently sourced material organized into draft units/chapters/sections)
  into a clean, publication-ready document: strips internal tracking
  artifacts (citation codes, confidence tags, process scaffolding) and
  sanitizes any private references, then runs a developmental edit
  (structure, ordering, argument soundness), a per-unit line edit against a
  derived house style, a whole-document consistency sweep, a scripted
  drift-verification pass, and a render + proofread. `assemble` runs the
  full pipeline. Use this whenever validated findings need to become
  something a reader outside the project could pick up and read, or to
  re-run just the editorial/render pass on an already-assembled draft.
  Never edits the source findings — reads them, writes a new, separate
  manuscript.
---

<!-- team-of-agents-generated: v1 -->

# Document Assembly

⚠️ **Experimental.** This skill is actively evolving — expect rough edges, and report issues if something breaks.

Shape: **pipeline** — assemble → sanitize → developmental edit → line edit
(gated on the first unit) → cross-unit consistency sweep → scripted
verification → render → proofread.

It exists because turning validated research into a real document is a
distinct job from validating the research in the first place, and doing it
carelessly can undo the validation work — a sloppy rewrite can quietly
change a claim, or a sanitization miss can leak a real name into something
meant for outside readers. This pipeline treats both as first-class,
checked steps, not an afterthought.

**The overriding constraint, repeated to every editorial agent verbatim:
edit how things are said, never what is said.** Every sentence traces to
evidence upstream. An editor may not change a claim, a number, an example's
substance, a citation's meaning, a sanitized name, or the heading structure.
When a sentence can't be improved without changing its meaning, it stays
as-is and gets flagged instead.

This is a **personal, cross-project skill** — usable on any assembled
document (a book, a report, a whitepaper), not tied to one format.
Vocabulary: a **unit** is one chapter/section/entry being assembled; the
**manuscript** is the assembled, in-progress document directory; **findings**
is whatever upstream material this pipeline reads from (typically
`corpus-mining`'s `findings.md` or an equivalent already-organized draft).

> **Path note (plugin install):** this file was written assuming a
> standalone install (`~/.claude/skills/document-assembly/` +
> `~/.claude/agents/`). If you installed this as a plugin instead, every
> `~/.claude/skills/document-assembly/...` path below means "the same-named
> folder bundled alongside this `SKILL.md`", and
> `~/.claude/agents/<name>.md` means "the matching file in this plugin's own
> `agents/` folder" — same relative layout, different root.

## Command routing

| Argument | Command |
|---|---|
| `assemble <findings> --target <output-dir>` | Full pipeline → `<output-dir>/manuscript/` — `references/assemble.md` |
| `edit <output-dir>` | Re-run just the editorial/render pass on an already-assembled manuscript |
| *(none)* | Ask where the validated findings are and where to assemble them |

## The team (first-class agents, installed globally at `~/.claude/agents/`)

| Agent | Role |
|-------|------|
| `document-assembler` | Phase 1-2: gathers and orders draft units, strips internal tracking artifacts, sanitizes private references. One instance per unit, run in parallel; the sanitization pass is a discover-then-rewrite two-stage sub-process (see `references/assemble.md`). |
| `document-editor` | Phase 3-5: derives (or applies) the house style sheet, runs the whole-manuscript developmental edit, the per-unit line edit, and the cross-unit consistency sweep. Invoked multiple times at different scopes within one pass — see `references/assemble.md`. |

> **How to invoke each role:** these are registered subagent types — launch
> each with `subagent_type: "<name>"`. If a name isn't available as a
> subagent type, fall back to a `general-purpose` agent and paste the role
> brief from `~/.claude/agents/<name>.md`.

## `assemble`

1. Resolve the findings source and the target output directory (ask if not
   given).
2. **Phase 1 — Gather and order.** Map validated findings to draft units per
   whatever table-of-contents / ordering document the project has. If a unit
   is missing, stop and say which one — never fabricate placeholder content.
3. **Phase 2 — Clean, enrich, sanitize.** One `document-assembler` instance
   per unit (parallel): strip internal tracking artifacts, convert inline
   citations to reader-facing endnotes, add a glossary/key-terms section.
   Then a two-stage sanitize pass (discover real names/orgs across every
   unit → build one cross-unit mapping → rewrite) before anything else
   touches the manuscript.
4. **Phase 3 — Developmental edit.** `document-editor`, whole-manuscript
   scope: structure, ordering, argument soundness, opening-promise /
   closing-payoff per unit. Diagnostic only — flags structural issues,
   doesn't restructure unilaterally.
5. **Phase 4 — Line edit, gated.** `document-editor`, one instance per unit
   needing an edit, against the derived house style. **Human gate**: show
   the first unit's before/after and get an explicit go-ahead before
   fanning out to the rest.
6. **Phase 5 — Cross-unit consistency sweep.** `document-editor`,
   whole-manuscript scope again: terminology drift, cross-references,
   duplicated explanations, voice drift, opening/closing re-check.
7. **Phase 6 — Scripted verification.** Deterministic checks, not agent
   judgment: no real name/org from the sanitization map survived; no
   internal ID pattern survived outside a notes section; heading structure
   unchanged from the pre-edit baseline; every citation marker has a
   matching note.
8. **Phase 7 — Render + proofread.** Render to the target format with
   whatever tool this project uses; then actually open the render and check
   structural navigation (TOC, bookmarks/links) and layout defects (orphaned
   headings, bad breaks) — don't declare done on the strength of the render
   command exiting cleanly.
9. Report back: which units were edited vs. skipped as unchanged, any
   developmental or cross-unit finding the user should weigh in on, and a
   closing qualitative read (writing quality / distinctiveness / execution —
   named categories, never a fabricated numeric score).

Full procedure — including the sanitization discover/rewrite mechanics, the
style-sheet derivation, the skip-clauses for re-runs, and the verification
script list — in `references/assemble.md`.

## Conventions

- **Never touch the source findings.** This pipeline reads validated
  findings and writes a new, separate manuscript directory. The source stays
  the untouched, fact-checked record.
- **Sanitize before editing, not after.** Editors should never see a real
  private name — sanitize in Phase 2, edit from Phase 3 onward.
- **Skip-clauses are deterministic, not vibes.** A unit that hasn't changed
  since its last completed edit-log entry doesn't need a re-edit; check this
  with a real content diff, not a "looks about the same" judgment call.
- **Memory**: `~/.claude/skills/document-assembly/memory/decisions.md`
  (standing calls) and a registry file (one row per completed pass — edition
  or version, date, style-sheet revision, findings count) so a future run
  can tell whether a manuscript has actually been through this pipeline
  before, or merely exists.

## Adding to the team

An optional adversarial "cold reader" pass (independent, blind first-reads
that stress-test the manuscript the way a real reader would, distinct from
an editor applying a style sheet) is a natural add-on — cost-gated, opt-in,
not part of the default pass. If your project wants it, model it as another
`document-editor` invocation with no style sheet in its prompt and a
"read this as a first-time reader" framing instead, triaged on convergence
(promote a finding only when 2+ independent instances flag the same
passage).
