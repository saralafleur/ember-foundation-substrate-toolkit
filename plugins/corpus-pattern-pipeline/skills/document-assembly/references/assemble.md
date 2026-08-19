# `assemble` — full procedure

## Phase 1 — Gather and order

Read the project's ordering document (a table of contents, an outline —
whatever names what belongs in the document and in what sequence) as the
binding spec. Map each unit to its source draft file. Skip anything that's
clearly process scaffolding (a fact-check report paired with a draft, a
superseded version) rather than actual content.

If a unit is missing, stop and tell the user which one and why you can't
proceed — don't silently skip it or fabricate placeholder content.

## Phase 2 — Clean, enrich, sanitize

For each unit, produce a new file in the manuscript directory that a reader
who has never seen the source project could read without hitting anything
that looks like an internal tracking artifact.

**Run this in parallel** — one `document-assembler` instance per unit; each
unit's cleaning is independent, and running them serially wastes wall-clock
for no benefit. Give each instance the exact source path, the target path,
and the transformation rules below verbatim (not paraphrased — consistent
output across many independently-run agents depends on identical
instructions).

**If a house style sheet already exists** (see Phase 3's derivation step —
normal after the first-ever assembly has been through this pipeline once),
include its full content in each Phase 2 instance's prompt so units are
generated already conforming to it, rather than relying on Phase 4 to
re-fix the same drift on every future regeneration.

### Transformation rules

1. **Convert inline citations to reader-facing notes, don't just delete
   them.** Replace an internal citation code with a note marker; write the
   note's actual content as something a general reader can understand — never
   expose a bare internal ID in reader-facing text.
2. **Strip research-process scaffolding entirely.** Confidence-tier jargon,
   internal tracking IDs, references to the project's own queue/backlog,
   dated correction annotations — none of it has reader-facing value.
3. **Keep genuinely reader-relevant caveats, reformatted.** If a unit
   legitimately needs to tell the reader "this is newer/less-tested
   material than the rest," say that once, plainly, in prose near the top —
   don't repeat an internal-looking disclaimer after every point.
4. **Keep all actual content.** Whatever the unit's real teaching/reporting
   content is, it stays — that's the document. Rename internal cross-
   reference labels to whatever a general reader would recognize (a plain
   title, not a bare internal id).
5. **Add a glossary/key-terms section** if the domain has jargon worth
   defining for a reader — pull 5-10 real terms the document itself relies
   on, not generic vocabulary. Keep definitions consistent with usage
   elsewhere in the unit.
6. **Preserve heading levels deliberately** — whatever your render tool uses
   to build a table of contents / bookmarks depends on consistent heading
   structure. Get this right per unit or navigation breaks.

### After all instances finish

Read every generated unit yourself and check for: leftover internal ID
patterns, inconsistent glossary definitions for a term used in more than one
unit, and any unit that got a treatment meant for a different unit. Fix
small inconsistencies directly rather than re-running a whole instance.

## Sanitization (two-stage: discover, then rewrite)

Don't try to discover and rewrite in one pass — the rewrite needs a
complete, deduplicated, cross-unit mapping first, since the same real
person/organization can appear in more than one unit and must get the same
fictional replacement everywhere for continuity.

**Stage A — Discover.** One instance per manuscript file (parallel), each
reporting every real private individual and real organization it finds.
Give every discovery instance an explicit always-keep list: the document's
own author/voice, any product/tool/vendor name (these are tools, not private
people or orgs), and any real, publicly-published external source cited for
their public work (citing a named public source is normal, not a privacy
issue).

**Stage B — Build the master mapping.** Read every discovery result
yourself and merge into one list — use role/context/timeframe details to
tell whether same-name mentions across units are the same real
person/entity or different ones who happen to share a name (never let two
distinct real entities collide onto one fictional identity). Assign each
distinct real person a fictional first name only (never a fictional last
name, never a real last name); assign each real organization a generic,
non-identifying description, not an invented fake name (a fake org name can
still be mistaken for a real reference; a bare role description can't).
Write this mapping to a file kept **outside** the manuscript directory — it
legitimately contains real names for internal reference and must never be
pulled into a render.

**Stage C — Rewrite.** One instance per *affected* file (parallel — skip
files discovery found nothing in), given the exact mapping, replacing every
occurrence (including in notes, not just body text) with its
fictional/generic replacement and nothing else.

**Re-scan after the rewrite** — grep the manuscript for every real
name/organization in the mapping, confirm zero occurrences, before moving
on.

## Phase 3 — House style sheet

If it doesn't exist yet, derive it by surveying the manuscript's own
dominant conventions in its hand-authored prose (not any machine-generated
boilerplate from Phase 2, which is uniform by construction and would skew
the survey). Codify: punctuation/number-style conventions, capitalization,
heading case, voice and person, contractions, list style, note tone, and an
explicit "AI-sounding prose" telltale list if the source material was
partly drafted with AI assistance (stock triadic-list constructions,
excessive signposting, uniform short-punchy-sentence cadence, stock
vocabulary).

If it already exists, don't rewrite it for this run; only propose an
addition if this run surfaces a real, recurring convention question it
doesn't already answer, and confirm any addition with the user before it
becomes binding for future runs.

## Phase 4 — Developmental edit, whole-manuscript

Run once per pass, before any per-unit line edit — restructuring after
line-editing wastes the line-edit work on prose that might move, merge, or
get cut. One `document-editor` instance reads the entire manuscript end to
end. Scope: unit ordering and pacing, argument soundness (does the evidence
actually support the claim being made, not just is it factually accurate),
opening-promise/closing-payoff per unit, and anything the document's own
front matter/outline promises that no unit delivers.

Diagnostic only — never rewrites prose. Write findings to a developmental-
edit findings file. Anything beyond a unit-local reordering suggestion gets
reported to the user as an open structural question, not resolved
unilaterally.

**Skip clause**: if a prior completed pass exists for this exact manuscript
and nothing structural changed since (check with an actual diff of the unit
list, not a guess), this step may be skipped and logged as such — unless an
existing unit was substantively rewritten in place, which a filename-level
diff can't see.

## Phase 5 — Line edit, gated on the first unit

Before spawning anything, check which units actually changed since their
last completed edit (a real content diff against the edit log, not a
changed-since-a-date guess) — skip anything unchanged, report which ones
were skipped.

One `document-editor` instance per unit needing an edit, given the style
sheet plus the overriding constraint above verbatim. Scope: grammar,
run-ons, paragraph hygiene, wordiness, AI-sounding-prose telltales,
mechanical consistency. Record every substantive edit (anything touching a
claim-carrying sentence) as a before/after pair in that unit's edit-log
entry.

**Human gate: edit the first unit first, alone.** Show the user a
representative before/after sample and get an explicit go-ahead before
fanning out to the rest — this calibrates editing aggressiveness once,
cheaply, instead of discovering after every unit that the edits were too
heavy- or light-handed. Fold whatever the user's reaction implies back into
the style sheet before the fan-out.

If a prior completed, human-approved pass exists for this exact manuscript
and the style sheet hasn't changed materially, the gate may be skipped — use
judgment; skip only when the calibration clearly still applies, gate again
if this pass's material is different enough in character that it might not.

## Phase 6 — Cross-unit consistency sweep

One `document-editor` instance, whole-manuscript scope, after Phase 5's
edits land. This is the second (with Phase 4) of only two steps that ever
see the whole document at once, so these defects are invisible to a
per-unit editor.

- **Terminology drift** — same concept, different names across units.
  Standardize to the style sheet's term.
- **Cross-references** — every internal reference resolves to the right
  unit, in both directions (a renamed unit heading that never propagated to
  the outline is invisible to a one-direction check).
- **Duplicated explanations** — the same concept taught from scratch twice.
  Trim the later occurrence to a brief reminder with a cross-reference when
  clearly redundant; report anything bigger rather than restructuring
  unilaterally.
- **Voice drift** — a unit that reads noticeably more formal/casual than its
  neighbors; flag, and retouch only if the fix is local.
- **Opening/closing re-check** — re-verify per unit after line editing,
  since a line edit can blunt an opening or trim a closing Phase 4 never
  saw in its pre-edit form.
- **Glossary/key-terms drift** — a definition inconsistency found and fixed
  here should also be noted as a candidate fix to Phase 2's generation
  rules, so it doesn't reproduce on the next regeneration.

If a prior sweep exists for this manuscript, read it and note for each new
finding whether it was already reported and is still open, rather than
re-surfacing a stale finding as new.

## Phase 7 — Scripted verification

Deterministic checks, not agent judgment — hand-checking these has a real
failure mode of silently skipping one under time pressure:

- Grep the manuscript for every real name/org in the sanitization mapping —
  expect zero occurrences.
- Grep for the internal ID pattern this project uses — expect zero
  occurrences outside a legitimate notes/citations section.
- Diff each unit's heading list (level, order, text) against a pre-edit
  baseline — expect an empty diff.
- Set-diff in-prose citation/note markers against defined notes per unit —
  expect zero orphaned markers, zero unmatched notes.

Any non-empty output here is a concrete violation, not a judgment call — act
on it directly.

## Phase 8 — Render and proofread

Render with whatever tool this project uses for its target format. Before
the first run, confirm any required tooling is installed and ask before
installing anything new.

Open the actual render and check, don't just assume:

- Structural navigation (table of contents, bookmarks/links) works and
  lands on the right unit.
- At least one citation/note reference actually resolves.
- No stray internal artifact survived into the rendered output — a text
  search alone isn't enough, since rendering can reflow text in ways a
  simple grep misses; look at a few actual rendered pages.
- Layout defects: orphaned headings, widows/orphans, bad breaks inside
  coined terms, notes splitting awkwardly, glossary entries splitting
  mid-definition.

Fix what's fixable in the source content; fix systemic layout issues in
whatever the render tool's styling layer is, and re-render. Report the
final size/length and tell the user plainly if anything looks off, rather
than declaring success on the strength of the render command exiting
cleanly.

## Registry

Keep one append-friendly table, one row per completed pass: manuscript
target, date, style-sheet revision in effect, developmental findings this
run, units edited vs. skipped, findings count, human-gate status. This is
what lets a future pass answer "was this actually done" without re-deriving
it from directory listings and file timestamps — a manuscript or its render
can exist without ever having been through this pipeline's actual editorial
steps.
