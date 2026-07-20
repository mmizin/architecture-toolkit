# Severity Levels — Shared Finding Taxonomy

The classification used for individual findings in any audit or review report
produced by this toolkit (`architecture-librarian`, `architecture-reviewer`,
or a skill's self-check). This is a different axis from
`references/review-checklist.md`'s verdict scale: verdict judges a whole
artifact; severity classifies one finding within a report.

## Kinds of finding

- **Conflict** — two artifacts state facts that cannot both be true (e.g. an
  accepted ADR and a C4 diagram name different technologies for the same
  component). Confirmed, not suspected.
- **Possible inconsistency** — artifacts disagree, but a plausible non-error
  explanation exists (migration in progress, deprecated doc not yet updated,
  intentional exception). Report it, don't call it a conflict. See the
  validation principle below.
- **Gap** — something required by project convention or by the artifact's own
  purpose is missing: a missing cross-link, an ADR with no status, a
  significant technology choice shown in a diagram with no ADR explaining
  it. Not every container or component needs an ADR — only flag a missing
  ADR where the diagram represents a real decision, not routine structure.
- **Recommendation** — no defect, but a concrete improvement (renumbering,
  clearer title, a cross-link that would help navigation).
- **Info** — worth noting, no action required.

## Severity tiers (apply to Conflict and Gap findings)

- **Critical** — breaks navigability or trust in the record itself (e.g. a
  circular supersede chain, two ADRs both numbered ADR-0007).
- **High** — accepted artifacts actively contradict each other.
- **Medium** — missing cross-link or index entry; doesn't mislead, but makes
  the record harder to trust or navigate.
- **Low** — cosmetic (title inconsistency, minor numbering gap).

## Validation principle

Do not assume every inconsistency is an error. Before reporting something as
a **Conflict**, rule out:

- a migration in progress (old and new artifacts briefly disagree on purpose)
- documentation that is simply stale, not wrong
- intentional, temporary divergence across branches
- an exception the team has already accepted elsewhere

When the explanation is unclear, report it as a **Possible inconsistency**
and say what's unclear, rather than asserting a conflict. Only escalate to
**Conflict** when there's no plausible legitimate explanation.

## Usage

- `architecture-librarian`: classify every finding by kind and severity;
  group the report by these categories.
- `architecture-reviewer`: use the same vocabulary for individual findings
  within a review, alongside its own overall verdict
  (`references/review-checklist.md`).
