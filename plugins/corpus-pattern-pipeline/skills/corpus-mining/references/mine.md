# `mine` — full procedure

## Extraction schema (`extracted/<source_id>.md`)

```markdown
---
source_id: S0123
relpath: <path from project root>
source_date: <YYYY-MM-DD or unknown>
participants: <role labels, not names, if this file will ever be read externally; real names OK internal-only per the privacy boundary>
category: <whatever stratification dimension matters for this corpus>
extraction_date: <today>
extractor_notes: <quality caveats: partial, off-topic, low-signal, etc.>
---

## Source context
What this source is, the setting, the problem/outcome it covers. 2-5 sentences.

## Observations
One block per distinct finding-worthy moment:

### OBS-<n>
- what_happened: <the concrete thing observed>
- observation_type: <pick categories that fit your domain>
- teaching_point / lesson: <the reusable takeaway, if any>
- candidate_pattern: <short memorable name, or none>
- counter_example: <what NOT to do, if the source shows one>
- generality: <EXACTLY one of: `durable` | `context-specific:<what narrows it>`>
- evidence: <short paraphrase or a short direct quote>
- locator: <timestamp, page, or anchor into the raw source — REQUIRED; auditing at scale is impossible without it>
- attribution: <`clear` | `reconstructed` — flag `reconstructed` whenever you had to infer who/what from context>

## Candidate patterns (rollup)
- <name> — one-line problem→solution — OBS refs

## Candidate counter-examples (rollup)
## Anything unusual or novel
## Dated / time-bound markers
Anything whose validity depends on a point in time (a tool version, a policy
that later changed, a capability that didn't exist yet).

## Outcome evidence
Any before/after, adoption, or impact material the source carries. Label
every item `self-reported` or `observed`. Don't let this evaporate into the
narrative summary above — if the point of mining this corpus is partly to
answer a value/impact question, this section is primary material for that,
not color.
```

### Citation-strength convention

Every citation in synthesis work carries a weight:
- `[T]` direct — the source explicitly states it
- `[P]` observed — happened in the source, not stated as a lesson
- `[A]` inferential — consistent with, but not stated
- `[E]` external — from an independently researched external source, if your
  project cross-references one (see "Adding to the team" in `SKILL.md`)

**Frequency counts `[T]` sources only** unless a specific run states otherwise.

### Source-independence rule (promotion gate)

Multi-source promotion requires independent evidence — genuinely different
sources, not the same person/entity's material appearing more than once. The
same individual demonstrating the same thing across multiple sessions counts
ONCE regardless of how many sources it appears in. Anything that could be
single-source doctrine needs `[E]` corroboration (or an equivalent
independent check) before promotion past `Recurring`.

## Pipeline layers

RAW (immutable sources) → NORMALIZED (manifest row: id/hash/metadata) →
EXTRACTED (structured per-source files; raw never reread except for targeted
validation) → CANDIDATES (synthesis/) → EVALUATED (audit/) → FINDINGS
(findings.md).

## Quality gates

- **Raw → Extracted**: extractor self-reports quality caveats; near-empty
  sources never enter (exclude mechanically via a manifest size/length
  field, not agent judgment). Every observation carries a locator.
- **Extracted → Candidate**: miner cites ≥1 source_id per claim with a
  citation-strength weight; single-`[T]`-source items stay `Observed`, not
  `Candidate`.
- **Candidate → Finding**: independent evaluator pass; upheld only with
  recurrence + independent support. Single-source-doctrine candidates
  additionally need external corroboration if your project has that check
  available.

## Batching a large corpus

For any corpus too large to extract in one pass, batch it — don't fire every
source's extraction agent at once:

1. Select a stratified batch (by whatever dimension matters — date range,
   category, source type) from `manifest.csv` (`status=pending`, above a
   minimum size threshold). Reserve a few longitudinal arcs (multiple
   sources from the same person/entity across time) if that's relevant to
   the corpus, then fill proportionally.
2. Run the batch's extraction fan-out, then mining, then evaluation over
   just that batch — or defer mining/evaluation until enough batches have
   landed to make cross-source pattern-finding worthwhile; either is fine,
   record which convention this project is using in `memory/decisions.md`.
3. Report yield (new candidates this batch) and cost (tokens) to the user
   before starting the next batch — this is a standing human gate, not a
   one-time approval. **Stop rule**: recommend pausing scale-up after 2
   consecutive batches yield fewer than 3 genuinely new candidates —
   diminishing returns is a real signal, not a reason to push through on
   inertia.

## Checkpointing / resume

Durable state = `manifest.csv` (status column), extraction files on disk,
`STATUS.md`, and a queue of open/blocked items if the project tracks one. On
resume: read `STATUS.md`, then the manifest's status counts, then continue
from pending items only. Completed extractions are detected by file
existence — never redone.

## Escalation

Retry with better context → try a different agent/approach → escalate to the
user. Escalate directly (don't keep retrying) for: unresolvable ambiguity in
what the corpus even means, a privacy-boundary judgment call, a decision
that would meaningfully change cost, or anything irreversible.

## Human decision packets

When a decision packet is needed (a batching stop-rule trigger, a privacy
judgment call, a scope question): state the **decision required**, **why it
matters**, the **options**, a **recommendation**, the **evidence** behind it,
your **confidence**, the **cost impact**, and **what happens if nothing is
decided**. This is the shape that lets a human answer in one read instead of
reconstructing context.
