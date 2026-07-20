# Terminology — Shared Glossary

Terms used consistently across all skills and agents in this plugin. If a
skill needs a term not listed here, add it here rather than defining it
locally.

- **ADR (Architecture Decision Record)** — a single significant decision,
  its context, and its consequences. See `references/adr-template.md`.
- **arc42** — a 12-section template for full architecture documentation. See
  `references/arc42-template.md`.
- **C4 model** — a four-level diagramming approach (Context, Container,
  Component, Code). See `references/c4-guidelines.md`.
- **Discovery report** — the output of `architecture-discovery`: system
  overview, component inventory, data flows, candidate ADRs, and doc gaps.
- **Candidate ADR** — a decision inferred from the code that has no ADR yet;
  surfaced by discovery, not yet accepted as documented.
- **Superseded** — an ADR's status once a later ADR replaces its decision.
  The old ADR is never deleted or rewritten; it's marked and linked forward.
- **Building block** — arc42's term for a static structural unit (module,
  service, component); maps to a C4 Container or Component.
- **Quality scenario** — a concrete, testable statement of a quality
  attribute (see `references/quality-attributes.md`) in context, e.g. "95th
  percentile checkout latency stays under 300ms at 10x current load."
- **One-way door / two-way door** — shorthand for a decision's reversibility:
  a one-way door is expensive or impossible to undo; a two-way door can be
  reversed cheaply. Used under Decision Drivers to flag how much rigor a
  decision deserves.
- **Architecture drift** — divergence between documented architecture (ADRs,
  C4, arc42) and the actual system's code/behavior. Detected by
  `architecture-discovery` re-scanning the repository, not by
  `architecture-librarian`, which only compares documentation artifacts
  against each other.
- **Artifact ownership** — each documentation type owns one kind of content;
  the others reference it rather than restate it. ADRs own the reasoning
  behind a decision (the "why"). C4 diagrams own the structural/relational
  view (the "what," visually). arc42 owns the narrative that ties everything
  together section by section (the "how it all fits"), citing ADRs and C4
  diagrams instead of repeating their content. Copying decision rationale or
  diagram content into arc42 (or vice versa) is a drift risk, not a
  convenience.
- **`ARCHITECTURE.md`** — the repository's architecture **entry point and
  navigation map**, owned by `architecture-bootstrap`. It holds links to the
  real artifacts plus a one-line summary of each, so a newcomer can find the
  documentation without knowing its layout. It explicitly does **not** hold
  architecture content: no arc42 sections (notably §3 Context and §4 Solution
  Strategy), no decision rationale, no diagrams. If a statement in
  `ARCHITECTURE.md` would be the source of truth for something, it belongs in
  an ADR, a C4 diagram, or an arc42 section instead, with `ARCHITECTURE.md`
  linking to it.
