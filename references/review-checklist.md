# Review Checklist — Shared Criteria

The single checklist used by `architecture-reviewer` (and by any skill doing
self-review before presenting an artifact to the user). Keeping this in one
place prevents ADRs, C4 diagrams, and arc42 sections from drifting toward
different, undocumented review bars.

## Applies to every artifact type

- **Significance** — is this actually worth documenting, or is it noise?
- **Problem clarity** — is the problem, and "why now," stated plainly before
  any solution appears?
- **Alternatives** — were realistic options evaluated, or is the chosen path
  presented as inevitable?
- **Trade-offs** — are downsides stated honestly, in terms of
  `references/quality-attributes.md`, not hand-waved?
- **Risks** — technical, operational, migration, vendor lock-in, maintenance.
- **Consistency** — does it conflict with or duplicate an existing ADR,
  diagram, or arc42 section? (See `architecture-librarian`.)
- **Durability** — would a new engineer understand the reasoning two years
  from now, without asking the author?

## Artifact-specific additions

- **ADR**: does every section of `references/adr-template.md` carry real
  content, not placeholder text? Is the status accurate?
- **C4 diagram**: single zoom level per diagram, every relationship labeled
  with intent + technology (see `references/c4-guidelines.md`)?
- **arc42 section**: does it reference ADRs/diagrams instead of restating
  them? Are unused sections marked "not applicable" rather than padded?

## Verdict scale

- **Accept** — ready as-is.
- **Accept with changes** — sound, but named gaps must be filled first.
- **Needs rework** — a core dimension above is missing or wrong; don't patch,
  redo.
