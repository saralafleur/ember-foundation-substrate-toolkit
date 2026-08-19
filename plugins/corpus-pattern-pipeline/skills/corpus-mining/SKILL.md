---
name: corpus-mining
argument-hint: "mine <path or description of the corpus>"
description: >
  Turns a large body of private, sensitive source material (transcripts,
  interviews, session logs, support tickets — call it "the corpus") into a
  validated set of findings: one structured extraction per source (a
  done-marker file, never re-extracted), cross-source candidate patterns
  mined with frequency and provenance, and an adversarial evaluation pass
  that only promotes what has independent, sourced support. `mine` runs the
  full pipeline (triage → parallel extraction fan-out → pattern mining →
  independent evaluation → lead synthesis) and writes a resumable set of
  findings other work (a document-assembly pass, a downstream research
  pipeline) can build on. The privacy boundary is absolute: raw source
  material and per-source extractions stay internal-only; nothing
  identifying ever reaches an external search or a reader-facing artifact.
---

<!-- team-of-agents-generated: v1 -->

# Corpus Mining

⚠️ **Experimental.** This skill is actively evolving — expect rough edges, and report issues if something breaks.

Shape: **pipeline** — triage → parallel per-source extraction fan-out →
pattern mining across extractions → independent adversarial evaluation →
lead synthesis into one findings file.

It exists because "just ask an agent to summarize this corpus" tends to
produce a plausible-sounding narrative that quietly generalizes from a
single source, loses provenance, and can't be audited later. This pipeline
keeps every claim traceable to the source(s) it came from, requires
independent (not merely repeated) support before a claim is promoted, and
runs an adversarial pass that is explicitly optimizing for correctness, not
agreement with the miner.

This is a **personal, cross-project skill** — usable from any
project/domain, not tied to one corpus type. Vocabulary: the **corpus** is
whatever large body of private source material you're mining; a **source**
is one item in it (one transcript, one ticket, one interview); an
**extraction** is that source's structured, done-marker summary; a
**candidate** is a pattern proposed across extractions, not yet validated; a
**finding** is a candidate that survived independent evaluation.

> **Path note (plugin install):** this file was written assuming a
> standalone install (`~/.claude/skills/corpus-mining/` +
> `~/.claude/agents/`). If you installed this as a plugin instead, every
> `~/.claude/skills/corpus-mining/...` path below means "the same-named
> folder bundled alongside this `SKILL.md`", and
> `~/.claude/agents/<name>.md` means "the matching file in this plugin's own
> `agents/` folder" — same relative layout, different root.

## Command routing

| Argument | Command |
|---|---|
| `mine <corpus>` | Full pipeline over `<corpus>` → `<output-dir>/findings.md` — `references/mine.md` |
| *(none)* | Ask where the corpus lives and where to save output |

## The team (first-class agents, installed globally at `~/.claude/agents/`)

| Agent | Role |
|-------|------|
| `corpus-triage` | Normalizes the corpus description into a structured brief, defines (or confirms) the extraction schema, sets the privacy rules, and inventories sources into a manifest. |
| `corpus-extractor` | Specialist. One instance per source, run in parallel. Reads exactly one source, writes one structured extraction file. Never reads another source; never re-extracts a source whose extraction already exists. |
| `pattern-miner` | Reads extraction files only (never raw sources) across the corpus; proposes candidate patterns with frequency counts and provenance (which source_ids support each one). |
| `pattern-evaluator` | Adversarially challenges every candidate AND the pipeline itself (extraction lossiness, provenance integrity). Optimizes for correctness, not agreement with the miner. |
| `pipeline-lead` | Synthesizes evaluator verdicts into `findings.md`, updates the manifest and `STATUS.md`, and owns the resume/checkpoint state. |

> **How to invoke each role:** these are registered subagent types — launch
> each with `subagent_type: "<name>"`. Always give the agent the corpus's
> output directory path. If a name isn't available as a subagent type, fall
> back to a `general-purpose` agent and paste the role brief from
> `~/.claude/agents/<name>.md`.

## `mine`

1. Resolve the corpus location and output directory (ask if not given).
   Default output location: `<project-root>/<output-dir>/` — pick a name
   that fits the project (`research/`, `knowledge/`, whatever's already in
   use there).
2. **Resume check** — if `<output-dir>/STATUS.md` already exists, read it
   first and ask the user whether to resume or start fresh. Never silently
   overwrite prior extraction work; extraction files on disk are the durable
   record of what's already done (checkpointing is by file existence, not a
   progress counter).
3. `corpus-triage` → `brief.md`, `manifest.csv` (one row per source: id,
   status, size, any stratification fields the corpus has — date, category,
   participant), `STATUS.md`.
4. **Parallel fan-out**: `corpus-extractor`, one instance per source that is
   `pending` in the manifest (skip anything already `extracted` — its file
   existing on disk is the done-marker), each writing
   `extracted/<source_id>.md`. For a large corpus, batch this in
   human-gated increments rather than firing every source at once — see
   "Batching a large corpus" below.
5. `pattern-miner` reads every extraction file → `synthesis/candidates.md`.
6. `pattern-evaluator` reads `candidates.md` + the extraction files →
   `audit/evaluation.md` (per-candidate verdict: upheld / weakened /
   rejected, plus any process findings about the pipeline itself).
7. `pipeline-lead` reconciles into `findings.md`, updates `manifest.csv`
   status and `STATUS.md`.
8. Report back in chat: how many sources were extracted this run, how many
   candidates were proposed, how many survived evaluation and at what
   confidence, and the path to `findings.md`.

Full procedure, including the extraction schema, the citation-strength and
source-independence conventions, and the batching/resume mechanics, in
`references/mine.md`.

## Privacy boundary

**This is the single most important rule in this pipeline — read it before
running anything.** The corpus is assumed private and potentially
identifying (real names, real organizations, real quoted material).

- **Internal files may keep identifying detail.** An extraction file
  (`extracted/<source_id>.md`) may name a real participant or organization
  when that's necessary to track provenance and independence — it never
  leaves the project's own filesystem.
- **Nothing external-facing may.** Any web search, any published or
  reader-facing artifact (a candidate summary shown to the user is fine; a
  document handed to someone outside the project is not, until it has been
  through a sanitization pass) must use role labels instead of names — "a
  support engineer at a client," never the person's actual name or employer.
- `corpus-triage` is the gate: it flags what's identifying in the brief so
  every later agent inherits the same boundary, rather than each one having
  to rediscover it independently.

## Conventions

- **Every claim needs a source_id.** `pattern-miner` cites which extraction(s)
  support each candidate; `pattern-evaluator` re-derives frequency counts
  from the underlying extractions rather than trusting the miner's count.
- **Independent support, not repetition.** A candidate promoted on the
  strength of "5 sources say this" is worthless if it's one person's
  material appearing five times. Require support from genuinely different,
  independent sources before promoting past `Observed`; a single-source
  candidate stays `Observed`, not `Candidate`.
- **Deterministic work is a script, not an agent.** Manifest updates,
  dedup, file-existence checks, near-empty-source exclusion — do these
  mechanically. Reserve agent calls for judgment work (extraction content,
  pattern proposal, evaluation).
- **Resumable by design.** `manifest.csv`'s status column, extraction files
  on disk, and `STATUS.md` are the durable memory for a mining run — a
  re-run picks up from pending sources only, never redoes completed
  extraction.
- **Memory**: `~/.claude/skills/corpus-mining/memory/decisions.md`
  (numbered, standing calls — read first) and `memory/cost-ledger.md`
  (append-only, one row per batch). Both updated by `pipeline-lead`.

## Adding to the team

A cross-reference against independently researched external material (does
the corpus's finding agree with, contradict, or extend what's publicly
known?) is a natural extension but isn't included as a first-class agent
here — see "What this plugin deliberately leaves out" in the top-level
README. If your project has a separate research pipeline installed, hand a
validated finding's topic to it directly rather than adding a new role to
this pipeline.
