# Brief Format — builders, scouts, runners

A brief is a self-contained work order. The subagent has none of your context: no conversation history, no plan, no second-brain. Everything it needs must be inside the brief. A vague brief produces confident garbage; the cost of a precise brief is always lower than the cost of arbitrating garbage.

## Builder brief template

```
ROLE: Builder (Opus). You implement exactly what this brief specifies — nothing more.

MISSION
<one paragraph: what this unit achieves and why it exists in the larger plan>

CONTEXT
<file paths the agent must READ ITSELF before coding (subagents have full
filesystem access), plus scout-report excerpts, schema facts, conventions,
brand/stack facts. Do NOT paste raw code — the brain works from reports, not
code; point at files and let the agent read them. Do not assume the agent can
infer project conventions — state them: language of code/comments (English),
commit style, test framework, error-handling conventions, etc.>

MANDATORY SKILLS
Before writing any code, read and follow: .claude/skills/ponytail/SKILL.md
<paste the resolved absolute path if the project uses a different location>.
Its rules bind all code you produce in this task. If the file does not exist
at the given path, report a BLOCKER — do not proceed without it.

SCOPE — files you MAY touch:
- path/one.ts
- path/two.ts
You may not create, modify, or delete any other file. If the task seems to
require touching another file, STOP that thread and report it as a blocker.

CONTRACT — your output must satisfy all of these:
1. <observable behavior #1>
2. <interface guarantee #2>
3. <tests that must pass / be added>
4. <performance / security constraint if any>

NON-GOALS
<explicitly list adjacent things NOT to do — refactors "while you're there",
dependency upgrades, style changes outside scope>

AMBIGUITY PROTOCOL
Do not resolve ambiguities unilaterally. If the brief underdetermines a decision,
pick the most conservative reading, flag it in your report under OPEN QUESTIONS,
and move on. Never ask interactive questions — you run to completion.

REPORT FORMAT (return exactly this structure)
- SUMMARY: what you did, 3–6 lines
- FILES CHANGED: list with one-line description each
- CONTRACT CHECK: for each contract item — met / not met / partially, with evidence
- OPEN QUESTIONS: ambiguities you flagged (empty if none)
- BLOCKERS: anything that prevented full completion (empty if none)
```

## Scout brief (read-only recon — Sonnet by default)

Same skeleton, but SCOPE is "read-only — you modify nothing", and the CONTRACT
is the questions to answer (e.g. "list every call site of syncCursor with file:line
and the surrounding transaction context"). Scouts return facts with file:line
citations, never opinions about what to build. Route scouts to Sonnet — faithful
reading is not judgment work; upgrade to Opus only when the recon itself requires
interpretation (implicit invariants, tangled architecture).

## Runner brief (Sonnet, mechanical)

Only for tasks where the spec fully determines the output. The CONTRACT is the
exact command(s) or transformation, and the report is raw results (full test
output, list of moved files). A runner brief that requires judgment is a routing
mistake — upgrade it to a builder brief on Opus.

## Dispatch checklist (verify before every launch)

- [ ] Brief is self-contained (an agent with zero context could execute it)
- [ ] Brief points at files, never pastes code/diffs (the brain must not have to read code to write it)
- [ ] Scope names exact files; no file appears in two parallel briefs
- [ ] Contract items are individually checkable (an adversary could test each)
- [ ] Non-goals prevent the two most likely scope-creep moves
- [ ] Model routing justified (judgment → Opus; fully-determined → Sonnet)

