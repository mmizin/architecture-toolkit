# Decision Drivers — Shared Checklist

The forces that commonly dominate an architectural decision. Not every driver
applies to every decision — pick the ones that are actually in tension for
the case at hand, and name them explicitly rather than leaving them implicit.

- **Business constraints** — deadlines, budget, contractual/compliance
  requirements, team size and skills available.
- **Existing architecture** — what's already in place that the decision must
  fit with, replace, or migrate away from.
- **Quality attributes in tension** — which attributes from
  `references/quality-attributes.md` are pulling in different directions
  (e.g., simplicity vs. scalability).
- **Team/organizational fit** — does the team have the skills to build and
  operate this? Is there a support/ownership model?
- **Time horizon** — is this a short-term fix or a long-term commitment? Does
  it need to survive a scale-up, a team handoff, a vendor change?
- **Reversibility** — how expensive is it to change this decision later
  (a "one-way door" vs. a "two-way door")?
- **External dependencies** — vendor stability, community/ecosystem health,
  licensing terms.

## Usage

- ADRs: list the drivers that actually applied under **Decision Drivers**
  (`references/adr-template.md`) — don't copy the whole list, select from it.
- Reviews: `architecture-reviewer` checks that the stated drivers explain the
  decision, and flags decisions where the real driver (e.g., a deadline) was
  left unstated in favor of a more flattering one.
