# Findings Contract v0.1 — Enforcement Rules

Rules `architecture-discovery` MUST enforce when producing or updating a
`FINDINGS.md`. Derived from the same discipline as
`references/PRD-CONTRACT-RULES.md` in the product toolkit: the template
defines the *shape*; this document defines the *rules* a generator would
otherwise violate while producing a document that looks complete.

`references/findings-template.md` defines the shape. Each rule below is
stated as a check, so it can serve directly as a review checklist for a
`FINDINGS.md` update.

## R-001: Term and namespace

- The artifact type is **Implementation Finding**, never bare "finding" —
  `severity-levels.md` already owns that term for a different taxonomy
  (documentation-consistency findings raised by `architecture-librarian` and
  `architecture-reviewer`). The two are never merged or cross-classified.
- ID prefix is `IF-`, sequential per `FINDINGS.md` file, zero-padded
  (`IF-001`). Never reused, never renumbered — an ADR or commit may cite an
  ID and that reference must stay resolvable.
- Deleting an Implementation Finding is not permitted. Closing or accepting
  it retires the ID; it does not free it (same rule as ADR numbering).

**Check:** every finding has an `IF-` ID; no ID appears twice; no gap has
been backfilled with a new finding.

**Open scope note:** "Implementation Finding" names discovery's current
domain — conditions observable in code and repository artifacts. It has not
been validated against non-code auditing (infrastructure, security,
operations). If a future auditing skill produces findings outside that
domain, revisit the name before reusing it — do not silently stretch
"implementation" to cover a condition it doesn't describe.

## R-002: Who creates a finding

- Only an auditing pass creates an Implementation Finding —
  `architecture-discovery` today. The `Source:` field exists so a future
  auditing skill *could* name itself without redefining the artifact type —
  it does not by itself authorize that skill to write to an existing
  `FINDINGS.md`. Adding a second `Source:` writer requires resolving the
  R-001/R-002 open scope notes (naming, ID namespace) first; until then,
  `Source:` has exactly one valid value: `architecture-discovery`.
- Authoring skills (`adr-expert`, `c4-expert`, `arc42-expert`,
  `architecture-bootstrap`) never create findings. They may reference an
  existing one (e.g., an ADR citing `IF-003` as the problem it resolves) but
  do not add new entries to `FINDINGS.md`.

**Check:** every finding's `Source:` names an auditing skill, not an
authoring one.

**Open scope note:** the `sequential per FINDINGS.md file` ID rule in R-001
assumes a single writer. It holds today because `architecture-discovery` is
the only auditing skill. If a second auditing skill (e.g. a future
security-audit) writes to the same artifact model, sequential allocation can
collide — resolve this before that happens, either with a shared ID
allocator or a per-source namespace (`AD-`, `SEC-`, ...). Not decided or
implemented now; do not add a second `Source:` writer to an existing
`FINDINGS.md` until it is.

## R-003: Evidence, not recommendation

- A finding states what was observed and the evidence for it. It never
  states what to do about it.
- Permitted content fields: `Observation`, `Evidence`, `Impact`,
  `Confidence`. No `Suggested fix`, `Recommendation`, or equivalent content
  field is permitted, in the body or as a template addition. This is a
  restriction on what a finding *says*, not on the entry's structure —
  identity/lifecycle metadata defined by `references/findings-template.md`
  (`Source`, `Status`, `Related`, `Reason`, `Accepted by`, `Resolved by`) is
  unaffected and governed by R-006/R-007 instead.
- `Impact` describes the plausible consequence of the observation (e.g.
  "concurrent signups with the same email may return HTTP 500"), stated as
  what could happen, not as an instruction.
- `Impact` is bounded by what `Evidence` directly supports, not by domain —
  a future security or operations audit may legitimately state a
  compliance or availability impact if the evidence supports it (e.g. "no
  backup verification job found" supporting "backup integrity is
  unverified" is in scope for that auditor). What's excluded is
  extrapolation beyond the evidence, not the domain the impact falls in:
  "may return HTTP 500 under concurrent requests" is directly supported by
  the code shown; "this will lose customers" is not — it requires business
  data discovery doesn't have access to, regardless of which auditor is
  writing it.

**Check:** no finding contains an imperative sentence ("should", "must",
"needs to") outside of quoting evidence; every `Impact` claim traces back to
something stated in `Evidence` — no claim requires information (business
metrics, user behavior, external systems) the evidence doesn't establish.

## R-004: No severity field

- Implementation Findings do not carry a `Severity` field.
  `severity-levels.md`'s tiers (Critical/High/Medium/Low) score
  documentation-trust impact and do not have an honest mapping to code-level
  risk — reusing the scale would misrepresent both.
- `Confidence` (`High` / `Medium` / `Low`) is the only structured judgment
  field, and it rates evidence strength (how directly the code supports the
  observation), never consequence severity.
- If code-risk severity scoring is needed later, it is a new, separately
  named scale — not an extension of `severity-levels.md` and not a silent
  reuse of its tier names.

**Check:** no finding has a `Severity` field; `Confidence` values are only
`High`, `Medium`, or `Low`.

## R-005: When a finding is created

An Implementation Finding is created when discovery observes code behavior
that carries a plausible, evidence-backed risk and that does not already
belong to one of discovery's other output categories.

**Check before filing:**
- Is this actually a **Candidate ADR** (a decision with no recorded
  rationale)? File it there instead — an unexplained choice is not the same
  as an observed risk.
- Is this a **Documentation Gap** (missing doc, not a code risk)? File it
  there instead.
- Is this routine/expected behavior with no observable risk, or a style
  preference? Not a finding — that's `architecture-reviewer` or lint
  territory, not discovery's.
- Can the observation be tied to specific evidence (file, line, or absence
  of an expected pattern)? An unfounded suspicion is not filed; note it to
  the user informally instead.

**Check:** every finding maps to exactly one primary artifact category —
never duplicated as both a Candidate ADR and an Implementation Finding, or
both a Documentation Gap and an Implementation Finding. ("Primary" leaves
room for a finding to carry secondary cross-references, e.g. `Related:
FC-001`, without that counting as belonging to a second category.)

## R-006: Status lifecycle

Three states only:

```
        ┌──────────────┐
        │     Open      │  (default, set by architecture-discovery)
        └───────┬──────┘
                │
      ┌─────────┴─────────┐
      ▼                    ▼
┌───────────┐        ┌───────────┐
│  Accepted │        │  Closed   │
└───────────┘        └───────────┘
  terminal              terminal
  Reason: +             Resolved by:
  Accepted by:          (ADR/commit/issue)
```

Both `Accepted` and `Closed` are terminal — neither transitions back to
`Open` or to each other. A regression against either is never an in-place
transition; it is always a *new* finding (R-007).

- `Open` (default) — observed, not yet resolved or explicitly accepted as
  risk.
- `Accepted` — the risk is acknowledged and deliberately not fixed.
  Requires a `Reason:` and an `Accepted by:` naming the deciding party;
  terminal unless new evidence contradicts it (see R-007).
- `Closed` — resolved. Requires `Resolved by:` naming the ADR ID, commit, or
  issue that resolved it.
- No `In Progress` state. Work-in-progress tracking belongs to the linked
  Issue/ADR, not to `FINDINGS.md` — a second copy of that state would go
  stale exactly the way `ARCHITECTURE.md` is barred from holding content it
  doesn't source (`references/terminology.md`).

**Check:** every finding's status is one of the three values; every
`Accepted` has a `Reason:` and an `Accepted by:`; every `Closed` has a
`Resolved by:`.

## R-007: Who changes status

- `architecture-discovery` never sets `Accepted` or `Closed` on a finding it
  created, even on a rescan. Discovery only creates and, on a rescan, notes
  whether the original evidence still holds.
- `Accepted` is set by the user, or by whichever skill turns the finding
  into a recorded decision (`adr-expert`, if the acceptance itself becomes
  an ADR).
- `Closed` is set by the user, or by `adr-expert` when the finding's
  resolution is the ADR just written, at the point the ADR is accepted.
- `Closed` and `Accepted` are terminal and immutable — a finding's status,
  once set, is never edited in place, and `architecture-discovery` never
  reopens one on a rescan. History stays honest: what was decided, and when,
  is never rewritten after the fact.
- If a rescan finds evidence contradicting a `Closed` or `Accepted` status
  (the original condition still reproduces), discovery files a **new**
  finding with the next `IF-` ID, and sets its `Related:` field to
  `Supersedes IF-0NN` naming the earlier one. It does not edit or reopen the
  earlier entry. The human decides what happens next (reopen the linked
  ADR/issue, file a fresh one, etc.) — discovery's job stops at reporting
  the regression with a traceable link back to what was previously decided.

**Check:** no finding transitions status without a named actor; no `Closed`
or `Accepted` finding is ever edited after the fact; every regression against
a resolved finding is filed as a new `IF-` entry carrying
`Related: Supersedes IF-0NN`, never as an in-place status change.

## R-008: Evidence must be reproducible

- `Evidence` names something a second reader can independently re-check —
  a file path (a line number or range when it adds precision, but not
  required — a stable pattern or path is often more durable than a line
  number that drifts on the next edit), an exact string/pattern that can be
  searched for, or a command whose output can be reproduced. A paraphrase of
  what the code does is not evidence.
- If the supporting evidence is an absence (no timeout, no `catch` block, no
  retry), name the specific location or scope searched, not just the
  conclusion — "no `catch` for `P2002` in `src/auth/register.ts:40-50`" is
  reproducible; "no error handling anywhere" is not.

**Check:** every `Evidence` field names a file, pattern, or command a second
reader could re-run to confirm the finding still holds; none states a
conclusion without a locator.

## Conformance

A `FINDINGS.md` conforms when all eight checks pass. Filling every field is
necessary but not sufficient — R-003, R-004, and R-005 are the rules a
generator will otherwise violate while producing a document that looks
complete: turning evidence into recommendations, inventing a severity score,
or duplicating a Candidate ADR as a finding.

Conformance says nothing about whether the findings are worth acting on.
That decision belongs to the user, an ADR, or a backlog item — never to
`architecture-discovery` itself.

Conformance also says nothing about completeness — it validates that what
was filed follows the artifact's discipline (evidence-based, correctly
categorized, correctly stated), not that discovery found every risk that
exists. A conformant `FINDINGS.md` with zero entries and a scan that missed
a real defect are indistinguishable by this checklist; catching the latter
is a discovery-quality question, not a contract-conformance one.
