# Adversarial Review — attack briefs and findings reports

The adversary's only job is to prove the builder wrong. It gets credit for valid
findings and for rigorous null results ("I attacked angles X, Y, Z and here is
specifically why each fails") — never for vague approval. "LGTM" from an adversary
is a defective report.

## Attack brief template

```
ROLE: Adversary (Opus). You do not fix anything. You attack this work to find
what is wrong with it. Assume the builder was competent but hurried, and that
at least one real problem exists — your job is to find it or prove it absent.

TARGET
<the list of changed files (from the builder's FILES CHANGED report) and, when
available, the exact command to see the change (e.g. `git diff <ref>` or
`git diff` in the worktree). READ THE CODE YOURSELF — open every changed file
and enough surrounding code to understand call sites. The brain does not paste
diffs; you have full filesystem access. You get the file list and the contract
only — NOT the builder's reasoning or report — form your own model of what the
code does.>

CONTRACT UNDER TEST
<paste the builder's contract verbatim. Every item is a claim to falsify.>

ATTACK SURFACE (work through ALL of these)
1. Contract violations — does the code actually guarantee each contract item?
   Trace the code paths; do not trust names or comments.
2. Edge cases — empty inputs, nulls, unicode/RTL, huge inputs, zero rows,
   duplicate keys, clock skew, timezone boundaries, pagination edges.
3. Concurrency & state — race conditions, double-submits, retries causing
   duplication, stale reads, missing idempotency, transaction boundaries.
4. Failure paths — what happens when the network call fails mid-way, the DB
   write succeeds but the queue publish fails, the webhook arrives twice?
5. Security & tenancy — injection, authz gaps, cross-tenant leakage, secrets
   in logs, unvalidated external input, SSRF via user-supplied URLs.
6. Data integrity — silent truncation, float money math, lossy migrations,
   sync-cursor gaps (records modified during a sync window), soft-delete leaks.
7. Tests — do the added tests actually assert the contract, or do they assert
   the implementation? What contract item has NO test?
8. Honesty of naming — functions/flags whose names promise more than the code does.
9. Coding-skill compliance — read the Ponytail skill referenced in this brief
   (.claude/skills/ponytail/SKILL.md unless another path is given) and check the
   diff against its rules. Each deviation is a finding (usually MEDIUM/LOW unless
   the rule protects correctness or security).

SPECIALIZATION (if assigned): <e.g. "focus attack surface 5–6 only, in depth">

REPORT FORMAT (return exactly this structure)
For each finding:
- ID: F1, F2, ...
- SEVERITY: CRITICAL (data loss / security / money) | HIGH (contract violation,
  user-visible bug) | MEDIUM (edge-case bug, missing test for contract item) |
  LOW (misleading naming, fragility)
- LOCATION: file:line
- CLAIM: one sentence — what is wrong
- REPRODUCTION: concrete input/sequence that triggers it (or code-path trace
  if not runnable)
- CONTRACT ITEM VIOLATED: number, or "implicit"
If no findings in some attack surface: one line per surface stating the strongest
attack you tried there and why it fails. A report with zero findings and zero
null-result explanations is invalid.
```

## Fable's arbitration rules

- Rule on every finding explicitly: VALID / INVALID / ACCEPTED RISK. Never
  silently drop one.
- INVALID requires a reason you could defend to the adversary ("F3 is wrong:
  the unique constraint on line 42 makes the duplicate impossible").
- A CRITICAL or HIGH finding can never be ACCEPTED RISK without stating it in
  the phase summary to the user.
- After fixes, the re-attack adversary gets the changed-file list of the fix,
  the ORIGINAL contract, and the list of fixed finding IDs (it reads the code
  itself). **Scope the re-attack:** verify each fixed finding + hunt for
  regressions in the surfaces adjacent to the change — not the full 9-surface
  sweep. Run the full sweep again only if the fix was large (new files, changed
  interfaces) or touched a high-stakes surface (security, money, data integrity).
- Two consecutive clean adversary passes are never required; one rigorous
  clean pass after fixes closes the unit.

## When to double up adversaries

Spawn two adversaries with disjoint SPECIALIZATION sections when the unit touches:
auth/permissions, billing/credits, multi-tenant queries, data migrations,
sync cursors / incremental sync, webhook ingestion, encryption/token handling.
One takes surfaces 1–4 (correctness), the other 5–6 + abuse thinking (security).

