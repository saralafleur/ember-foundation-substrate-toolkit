# ember-foundation-substrate-toolkit

Ember Foundation Substrate's idea-evaluation pipeline for Claude Code, packaged
as an installable plugin: three chained virtual teams that take a product idea
from question to plan.

| Skill | Question it answers | Output |
|---|---|---|
| `team-research` | What does the external evidence say about this space? | `research/<topic-slug>/dossier.md` — sourced market size, competitors, monetization patterns, contradicting evidence |
| `team-viability` | Should we build (or keep investing in) this? | `viability/<target-slug>/verdict.md` — **BUILD / DON'T BUILD / NEEDS MORE EVIDENCE** |
| `team-business-plan` | It's a BUILD — what's the plan? | `viability/<target-slug>/business-plan/business-plan.md` — offering & MVP, financials, plan-level go-to-market, milestones, risks |

See [the plugin's README](plugins/ember-foundation-substrate-toolkit/README.md)
for how the chain works and the full agent roster.

## Install (Claude Code)

**1. Add this marketplace** — from GitHub:

```
/plugin marketplace add saralafleur/ember-foundation-substrate-toolkit
```

Or from a local clone:

```
/plugin marketplace add /path/to/ember-foundation-substrate-toolkit
```

**2. Install the plugin:**

```
/plugin install ember-foundation-substrate-toolkit@ember-foundation-substrate
```

**3. Reload** — run `/reload-plugins` to activate immediately, or restart your
Claude Code session.

## Maintaining

The working copies live in the maintainer's `~/.claude/skills` and
`~/.claude/agents`; this repo is the packaged distribution. Run
`python3 scripts/sync-from-local.py` to re-import them (applies the
experimental banner, sanitizes personal wording, never ships memory), review
`git diff`, then push.
