# Review Checklist — Shared Criteria

The single checklist used by `architecture-reviewer` (and by any skill doing
self-review before presenting an artifact to the user). Keeping this in one
place prevents ADRs, C4 diagrams, and arc42 sections from drifting toward
different, undocumented review bars.

## Applies to every artifact type

- **Significance** — is this actually worth documenting, or is it noise?
- **Problem clarity** — is the problem, and "why now," stated plainly before
  any solution appears?
- **Decision drivers** — are the quality attributes and constraints that
  drove the decision explicitly identified (see
  `references/quality-attributes.md` and `references/decision-drivers.md`),
  or left implicit?
- **Alternatives** — were realistic options evaluated, or is the chosen path
  presented as inevitable?
- **Assumptions** — are important assumptions stated explicitly, or does the
  reasoning rest on implicit knowledge a reader wouldn't have?
- **Evidence** — are claims supported by benchmarks, incidents, requirements,
  standards, or other verifiable evidence where it matters, or just asserted?
- **Trade-offs** — are downsides stated honestly, in terms of
  `references/quality-attributes.md`, not hand-waved?
- **Risks** — technical, operational, migration, vendor lock-in, maintenance.
- **Consequences** — are long-term positive *and* negative effects explicit?
- **Durability** — would a new engineer understand the reasoning two years
  from now, without asking the author?
- **Consistency** — does it contradict an existing ADR, diagram, or arc42
  section in a way that materially affects its validity? Check only the
  directly related artifacts you already have in hand (the ones it cites, or
  the 2-3 most obviously relevant) — do not go searching the repository for
  more. If verifying consistency would mean scanning beyond that, stop and
  note it as an open question or hand it to `architecture-librarian` instead
  of doing an ad hoc repository-wide audit.

## Artifact-specific additions

- **ADR**: does every section of `references/adr-template.md` carry real
  content, not placeholder text? Is the status accurate?
- **C4 diagram**: single zoom level per diagram, no mixed levels; every
  relationship labeled with intent + technology; the right abstraction level
  for the audience; element count manageable (split the diagram if it has
  become unreadable). See `references/c4-guidelines.md` for the conventions
  and Mermaid examples these criteria check against. Whether the diagram still
  matches the *real system* is not reviewable from the diagram alone — hand
  that to `architecture-discovery` rather than cross-checking source code
  yourself.
- **arc42 section**: does it reference ADRs/diagrams instead of restating
  them? Are unused sections marked "not applicable" rather than padded?

## Verdict scale

- **Accept** — ready as-is.
- **Accept with changes** — sound, but named gaps must be filled first.
- **Needs rework** — a core dimension above is missing or wrong; don't patch,
  redo.
