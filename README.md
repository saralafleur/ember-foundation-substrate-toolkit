# ember-foundation-substrate-toolkit

A Claude Code plugin marketplace with two independent plugins.

## ember-foundation-substrate-toolkit — idea-evaluation pipeline

Three chained virtual teams that take a product idea from question to plan.

| Skill | Question it answers | Output |
|---|---|---|
| `team-research` | What does the external evidence say about this space? | `research/<topic-slug>/dossier.md` — sourced market size, competitors, monetization patterns, contradicting evidence |
| `team-viability` | Should we build (or keep investing in) this? | `viability/<target-slug>/verdict.md` — **BUILD / DON'T BUILD / NEEDS MORE EVIDENCE** |
| `team-business-plan` | It's a BUILD — what's the plan? | `viability/<target-slug>/business-plan/business-plan.md` — offering & MVP, financials, plan-level go-to-market, milestones, risks |

See [the plugin's README](plugins/ember-foundation-substrate-toolkit/README.md)
for how the chain works and the full agent roster.

## corpus-pattern-pipeline — private-corpus mining and document assembly

Two chained pipelines: turn a large body of private source material into
validated, sourced findings, then assemble those findings into a polished,
sanitized document.

| Skill | Question it answers | Output |
|---|---|---|
| `corpus-mining` | What does this corpus actually teach, and is it real or a single anecdote? | `<output-dir>/findings.md` — validated, sourced candidate patterns |
| `document-assembly` | How do I turn validated findings into something publishable? | `<output-dir>/manuscript/` — clean, sanitized, house-style-edited, rendered document |

See [the plugin's README](plugins/corpus-pattern-pipeline/README.md) for the
privacy boundary this pipeline is built around and the full agent roster.

## Install (Claude Code)

**1. Add this marketplace** — from GitHub:

```
/plugin marketplace add saralafleur/ember-foundation-substrate-toolkit
```

Or from a local clone:

```
/plugin marketplace add /path/to/ember-foundation-substrate-toolkit
```

**2. Install a plugin:**

```
/plugin install ember-foundation-substrate-toolkit@ember-foundation-substrate
/plugin install corpus-pattern-pipeline@ember-foundation-substrate
```

**3. Reload** — run `/reload-plugins` to activate immediately, or restart your
Claude Code session.

## Maintaining

**`ember-foundation-substrate-toolkit`**: the working copies live in the
maintainer's `~/.claude/skills` and `~/.claude/agents`; this repo is the
packaged distribution. Run `python3 scripts/sync-from-local.py` to re-import
them (applies the experimental banner, sanitizes personal wording, never
ships memory), review `git diff`, then push.

**`corpus-pattern-pipeline`**: hand-maintained directly in this repo — it
was genericized from a project-scoped (not `~/.claude`-global) pipeline, so
`sync-from-local.py` doesn't touch it. Edit its files under
`plugins/corpus-pattern-pipeline/` directly, review `git diff`, then push.
