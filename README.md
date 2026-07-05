# fable-brain

A Claude Code skill that turns your session into an **orchestrator brain** that never writes code itself — it plans, dispatches builder subagents to implement, then dispatches independent **adversarial** subagents whose only job is to attack the builders' work and find bugs, contract violations, and security holes. The brain arbitrates the findings and closes the loop.

## Why

A single agent reviewing its own code shares its own blind spots. Splitting **builders** (who construct) from **adversaries** (who are rewarded only for finding what's wrong) creates genuinely independent verification. The orchestrator stays cheap: it reads reports, not code — all high-volume reading and writing is delegated.

## The loop

1. **Recon** — Sonnet scouts read the code and report facts with file:line citations. The brain reads reports, not files.
2. **Plan** → user approval gate. Decomposition, dependency graph, agent roster, explicit output contracts per unit.
3. **Build** — Opus builders implement from self-contained briefs with exact file scope. No improvisation.
4. **Attack** — Opus adversaries read the changed code themselves and hunt for bugs across a 9-surface attack list. "Looks fine" is a failure mode; clean surfaces require null-result explanations.
5. **Arbitrate** — the brain rules each finding valid / invalid / accepted-risk from the reports, dispatches fix briefs, re-attacks scoped. Capped at 3 fix loops per unit.
6. **Integrate & summarize** → user approval gate.

## Install

Install both skills (fable-brain + its bundled coding-standards skill, Ponytail):

```bash
npx skills add roeeglov/fable-brain@fable-brain -g
npx skills add roeeglov/fable-brain@ponytail -g
```

Or manually: copy the `fable-brain/` and `ponytail/` folders into `~/.claude/skills/` (global) or your project's `.claude/skills/`.

**Optional:** copy `fable-brain/agents/*.md` into your project's `.claude/agents/` so the Task tool can target `opus-builder`, `opus-adversary`, and `sonnet-runner` by name with the right model pinned. The skill's `agents/` folder is the source of truth — re-copy on updates rather than editing project copies.

## Bundled: Ponytail (coding-standards skill)

Briefs instruct every code-writing agent to load `.claude/skills/ponytail/SKILL.md` — a minimal-code / anti-over-engineering skill ("the laziest solution that actually works") — before writing anything, and adversaries audit the diff against its rules. **Ponytail is included in this repo** (MIT), so installing both folders gives you a working setup out of the box.

Already have your own coding-standards skill? Either:

- swap the path in `fable-brain/SKILL.md`, `references/briefing-format.md`, and `references/adversarial-review.md` for yours, **or**
- remove the MANDATORY SKILLS section from the brief templates.

If the referenced skill file is missing at run time, builders report a BLOCKER (by design — the dependency fails loud, not silent).

## Usage

The skill is **explicit-invocation only** — it will not auto-activate on ordinary tasks (it spawns many agents and is deliberately token-expensive). Trigger it by saying:

- `use fable brain` / `fable brain` / `מוח`
- `run this through the brain`
- `dispatch agents`
- `builder + breaker`

## Layout

```
fable-brain/
├── SKILL.md                        # the orchestrator protocol
├── references/
│   ├── briefing-format.md          # builder / scout / runner brief templates
│   └── adversarial-review.md       # attack briefs, severity scale, arbitration rules
└── agents/
    ├── opus-builder.md             # implementation agent (model: opus)
    ├── opus-adversary.md           # red-team agent (model: opus)
    └── sonnet-runner.md            # mechanical execution agent (model: sonnet)
ponytail/
└── SKILL.md                        # bundled coding-standards skill (MIT) — builders follow it, adversaries audit against it
```

## Design principles

- **The brain never implements.** All code is written by dispatched builders.
- **Reports, not code.** The brain reads builder summaries and adversary findings — never raw diffs. Briefs point at files; agents read them themselves.
- **Adversaries never fix.** They return structured findings (severity, location, reproduction, contract item violated) and nothing else.
- **Judgment → Opus, mechanical → Sonnet.** If you hesitate about routing, that hesitation means Opus.
- **Bounded loops.** 3 fix iterations per unit, then escalate to the user instead of burning agents.

## License

MIT
