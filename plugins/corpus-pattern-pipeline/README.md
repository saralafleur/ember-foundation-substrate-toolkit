# corpus-pattern-pipeline

⚠️ **Experimental.** These skills are actively evolving — expect rough edges,
and report issues if something breaks.

Two chained pipelines for turning a large body of **private, sensitive
source material** — transcripts, interviews, support tickets, session logs,
whatever a project's real corpus is — into a validated body of findings, and
then into a polished document a stranger could read without ever seeing the
private source.

| Skill | Question it answers | Output |
|---|---|---|
| `corpus-mining` | What does this corpus actually teach, and is it real or a single anecdote? | `<output-dir>/synthesis/candidates.md` (mined) → `<output-dir>/audit/evaluation.md` (adversarially checked) → `<output-dir>/findings.md` (validated, sourced) |
| `document-assembly` | How do I turn validated findings into something publishable? | `<output-dir>/manuscript/` — clean, sanitized, house-style-edited units plus a rendered final document |

## The chain is the point

- `document-assembly` is meant to consume `corpus-mining`'s validated
  `findings.md` (or any equivalently fact-checked source material) — it
  never re-derives claims, it only assembles and edits prose that already
  traces to evidence.
- Every stage in `corpus-mining` keeps the **privacy boundary** absolute:
  raw source material and its structured extraction stay internal-only;
  nothing with an identifying detail is ever put in front of an external
  search or allowed into a reader-facing artifact. See "Privacy boundary"
  in `skills/corpus-mining/SKILL.md` — this is the single most important
  invariant in the whole plugin, not a nice-to-have.
- `document-assembly`'s editorial pass is bound by one rule repeated to
  every editorial agent verbatim: **edit how things are said, never what is
  said.** Every sentence traces to evidence upstream; an editor may improve
  wording, never a claim.

## Usage

```
/corpus-mining mine <path to the corpus, or a description of where it lives>
/document-assembly assemble <path to validated findings> --target <output-dir>
```

Outputs land in the current project (`<output-dir>/` at the project root,
your choice of name — e.g. `research/` or `knowledge/`). Team memory
(run-logs, standing decisions) lives with the plugin's bundled skill
folders — see the "Path note (plugin install)" in each `SKILL.md`.

## Agents

7 registered subagent types: `corpus-triage`, `corpus-extractor`,
`pattern-miner`, `pattern-evaluator`, `pipeline-lead`, `document-assembler`,
`document-editor`.

## What this plugin deliberately leaves out

This is a generalized extraction of a real, much more elaborate pipeline
that was built for one specific project over months of real runs. Left out
on purpose, because it didn't generalize cleanly or wasn't load-bearing for
a first cut of this plugin:

- **Per-edition style-sheet variants and a privacy-lock/veil integration**
  for a document deliberately modeled on a named external reference work
  (box names, page composition, trade-dress risk). If your project needs
  this, treat `document-assembly`'s style-sheet step as the extension point.
- **A specific external-corroboration agent role** (comparing corpus
  findings against independently researched external sources). If your
  project also has a research pipeline installed, hand a validated
  candidate's topic to it and treat the result as `[E]`-weight corroborating
  evidence per `corpus-mining`'s citation-strength convention — see
  `references/mine.md`.
- **A specific rendering toolchain.** `document-assembly` describes the
  render/proofread stage generically; wire in whatever tool actually
  produces your target format (PDF, static site, slide deck).
