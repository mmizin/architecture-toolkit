# Delta Review

Governance reference. Defines the Delta Review capability: a bounded review
mode any reviewer subagent can adopt to re-check an artifact after a fix
cycle without re-deriving the whole review from scratch. Today this is
`architecture-reviewer` (mirrored in product-toolkit as `product-reviewer`);
the contract is not specific to either agent or to any one artifact type
(ADR, PRD, arc42 section, discovery report, or others to come) — it assumes
only that the reviewer produces findings with stable identifiers, per
`FINDINGS-CONTRACT.md` / the product-toolkit equivalent.

## Purpose

Delta Review minimizes the cost of iterative review cycles while preserving
the independence and confidence of the review process. It is an optimization
of the review workflow, not a replacement for Full Review.

## Problem this solves

The default review mode (Full Review) re-reads and re-judges an entire
artifact every time it runs. In a Review → Fix → Review → Fix loop, each pass
pays the full read-and-reason cost again, even though only a small, known
region actually changed. Delta Review and Self-Verification exist to break
that loop without weakening the independent check Full Review provides.

## Review pipeline

A review cycle has four possible steps. Not all are used in every cycle. The
linear form below names the steps; it is not the control flow — Fix,
Self-Verification, and Delta Review typically repeat several times before
Final Review runs once, at the end:

```
Full Review  →  Fix  →  Self-Verification  →  Fix (if needed)  →  Delta Review  →  Final Review
```

The actual control flow loops between Fix, Self-Verification, and Delta
Review until the orchestrator judges the artifact ready for sign-off, or an
escalation trigger fires:

```
Full Review
    │
    ▼
   Fix ──► Self-Verification ──► Delta Review
    ▲                                 │
    │        (more fixes needed)      │
    └─────────────────────────────────┤
                                       │ (orchestrator decides ready
                                       │  for final review)
                                       ▼
                                 Final Review
                                       │
                                       ▼
                                  Sign-off
```

Any step in the loop may instead trigger escalation to Full Review, per the
Escalation clause below — escalation is not a normal exit from the loop, it
is an abort of it.

- **Full Review** — the existing, only-mode-today behavior. Independent
  subagent, reads the whole artifact, produces findings from scratch. Used for
  the first pass on an artifact.
- **Self-Verification** — the orchestrator (not the reviewer subagent) checks
  its own fix against the specific finding it claims to resolve. No new model
  invocation, no re-read of the full artifact.
- **Delta Review** — the reviewer subagent, invoked again, but scoped to prior
  findings, claimed fixes, and changed regions only. Independent, but bounded.
- **Final Review** — the final independent review before sign-off. Today
  this role is fulfilled by a Full Review, invoked once, deliberately, when
  the orchestrator judges the artifact is ready. The pipeline defines Final
  Review by its role, not by a fixed implementation — a different review
  mode may fill this role in the future without changing this contract.

Self-Verification and Delta Review are not interchangeable: Self-Verification
answers "did I do what I claimed," performed by the party with a conflict of
interest, so it cannot be the basis for sign-off. Delta Review answers "is
what was claimed actually correct and complete," performed by the independent
reviewer, so it can.

## Preconditions

Delta Review may only be invoked when all of the following hold. If any is
missing, the orchestrator runs a Full Review instead.

- A previous Full Review (or a prior Delta Review) exists for the artifact,
  with recorded findings.
- Those findings have stable identifiers to reference.
- Claimed fixes are explicitly mapped to the finding IDs they address — the
  reviewer is never asked to guess which change addresses which finding.
- The changed region is known and statable as a bounded set of files or
  sections.
- The prior findings are still applicable — the artifact has not changed, in
  the specific regions the findings concern, such that those findings no
  longer describe the current state. This is a narrower, finding-specific
  check than the changed-region precondition above; a large rewrite usually
  fails both at once, but a small, targeted edit can invalidate a finding's
  premise (e.g., the section it discusses was deleted) without making the
  overall changed region unbounded. Either failure alone requires a Full
  Review.

## Escalation: when Delta Review is not enough

Delta Review escalates back to Full Review when:

- the changed region cannot be described as a bounded set of sections/files
  (e.g., a restructure touching most of the document), or
- Delta Review itself surfaces a new finding at Critical/High severity
  (severity scale per `severity-levels.md`).

Beyond these two cases, escalation is a judgment call for the orchestrator,
informed by how many Delta Review cycles have run against the artifact
without reaching sign-off and by the reviewer's own escalation
recommendations (see Scope boundary, below). This spec does not fix a cycle
count or other numeric threshold — that is a heuristic to derive from usage,
not to freeze into the contract before the capability has been used.

## Delta Review — contract

**Responsible for:** verifying claimed fixes against prior findings, and
surfacing new issues introduced within the changed region.

**Not responsible for:** re-deriving findings the prior Full Review already
accepted as resolved, or judging (rather than flagging for escalation)
regions outside the stated changed set — resolving those is Full Review's
job, per the Escalation clause above and the Scope boundary below.

### Inputs

| Field | Description |
|---|---|
| Prior Findings | The finding set from the last Full or Delta Review, each with a stable ID (e.g., `FC-001`). |
| Claimed Fixes | A mapping of finding ID → what changed to address it, stated by whoever applied the fix. |
| Changed Region | The file(s) and, within them, the section(s) or diff hunks that changed. This is the scope boundary (see below) — it is supplied by the caller, not inferred by the reviewer. |

### Outputs

Two separate dimensions — a per-finding verdict, and a process-level review
outcome. These are not the same axis; a Delta Review always produces the
former, and may additionally produce the latter.

**Finding verdict** (per prior finding):

- **Resolved** — the claimed fix addresses the finding as stated.
- **Partially Resolved** — the fix addresses part of the finding; the
  remainder is restated as a finding.
- **Not Resolved** — the claimed fix does not address the finding; the
  original finding stands.

Plus, independent of any finding's verdict:

- **New findings in changed region** — issues the reviewer observes that were
  introduced by, or newly visible because of, the changes in the stated
  Changed Region only.

**Review outcome** (process-level, not a finding property):

- **Continue Delta Review** — the default; no escalation trigger fired.
- **Escalate to Full Review** — the reviewer cannot verify one or more
  findings within the bounded scope (e.g., a fix's correctness genuinely
  depends on content outside the Changed Region), or a
  Preconditions/Escalation trigger applies. Carries a reason. This can
  accompany any finding verdict above — e.g., a finding may be "Not Resolved"
  *and* the review outcome may be "Escalate" if the reviewer also can't fully
  assess it within scope. The orchestrator resolves an Escalate outcome by
  widening scope or running a Full Review, per the Escalation clause above.

### Scope boundary

The reviewer evaluates only:

1. the specific findings and fixes named in the input, and
2. the content inside the stated Changed Region.

The reviewer does not go looking for issues in files/sections outside the
Changed Region — that active search is out of scope for this pass by
construction, not by the reviewer's judgment call. But if, while reviewing
the declared region, the reviewer observes a high-confidence concern outside
it that is a direct consequence of the changes under review (e.g., a changed
term now contradicts an existing finding or requirement elsewhere), it may
recommend escalation to Full Review rather than silently ignoring it. This is
a recommendation, not a new finding raised against the out-of-scope region —
the reviewer does not judge or resolve that issue itself; it flags it, and
the orchestrator decides whether to widen the region or escalate.

A finding not in the Prior Findings input and not inside the Changed Region is
never resolved, judged, or closed by a Delta Review, regardless of severity —
at most it is flagged for escalation, per the paragraph above.

## Self-Verification — contract

**Responsible for:** the orchestrator confirming, without a new reviewer
invocation, that the intended change was actually made and appears to
address the targeted finding.

**Not responsible for:** independently validating that the finding is fully
resolved — that determination belongs to Delta Review or Full Review, never
to Self-Verification. Self-Verification is a self-check by the party with a
conflict of interest; it cannot mark a finding "Resolved."

### Inputs

- The single finding being addressed.
- The specific edit made in response to it.

### Outputs

- **Ready for Delta Review** / **Needs further changes** — an informal,
  orchestrator-only readiness judgment, not a review finding, and not
  recorded in the artifact's finding history the way Delta/Full Review
  outputs are.

## Relationship to existing reference files

- Severity of new findings raised in Delta Review uses `severity-levels.md`,
  unchanged.
- The review criteria a Delta Review checks against are the same criteria in
  `review-checklist.md` (architecture-toolkit) or
  `product-review-checklist.md` (product-toolkit) — Delta Review does not
  introduce new criteria, only a bounded application of the existing ones.
  The citation is bidirectional: `review-checklist.md` notes that it is
  applied in full by Full Review and in bounded form by Delta Review.
- Listed in the README's reference index, per `plugin-design-principles.md`'s
  rule that every reference must be listed there and have at least one
  inbound citation.