---
name: c4-expert
description: >-
  Create and review C4 model diagrams (System Context, Container, Component,
  and optionally Code) for a software system, from provided architecture
  information — does not reconstruct architecture from code. Produces
  diagrams as text (Mermaid or Structurizr DSL) suitable for
  version-controlled architecture documentation. Use when the user wants to
  visualize system architecture, document containers and their interactions,
  or keep diagrams consistent with documented system boundaries. Works from
  an architecture-discovery report or from a description the user provides.
---

# C4 Expert

> Status: Initial implementation — structure and workflow are defined; deepen
> during use.

Model software architecture with the C4 approach: a small hierarchy of diagrams
at increasing zoom levels, each with a clear audience.

## Scope

**Responsible for:**
- creating and maintaining C4 diagrams
- choosing the appropriate C4 abstraction level for the audience
- documenting system context, containers, and components visually
- validating a diagram's own structure against C4 conventions
- **supplementary diagram types** — Dynamic (runtime collaboration for a
  scenario) and Deployment (mapping containers to infrastructure nodes). These
  are not levels; they complement the four below. *v0.1: ownership declared,
  conventions and Mermaid examples land in v0.2 — expect scope, not yet
  step-by-step guidance.*

**Not responsible for:**
- discovering architecture or system boundaries from source code
- judging whether the underlying architecture is a good design
- creating ADRs or documenting decision rationale
- repository-wide documentation consistency audits

**Escalates to:**
- `architecture-discovery` — when system boundaries or components are
  unknown and need to be reconstructed from code, rather than assumed.
- `architecture-reviewer` — when the question is whether the architecture
  the diagram represents is good, not whether the diagram is well-formed.
- `architecture-librarian` — when a diagram conflicts with an ADR, an arc42
  section, or another diagram elsewhere in the repository.
- `arc42-expert` — when the architecture needs narrative documentation
  beyond what a diagram can show.

## When to use

- Documenting a system's context and container structure.
- Visualizing how services, data stores, and external systems interact.
- Reviewing an existing diagram for C4 compliance and appropriate
  abstraction — not for whether the architecture it depicts is sound.

## The four levels (use only what the audience needs)

1. **System Context** — the system as one box, its users, and neighbouring
   systems. For everyone.
2. **Container** — deployable/runnable units (apps, services, databases) and
   their interactions. For technical stakeholders. *Usually the most valuable.*
3. **Component** — components inside one container. Only where it earns its
   keep; avoid a component diagram spanning an entire system — scope it to
   one container.
4. **Code** — class-level detail. Usually generated from code rather than
   maintained by hand; don't create one unless the user explicitly needs
   this level.

Diagrams show structure and relationships only — see
`references/terminology.md` for artifact ownership conventions: C4 owns
structural views; ADRs own decision rationale; arc42 owns the narrative
that ties them together.

## Workflow

1. Establish scope and audience — which level(s) are actually needed.
2. Identify, using only provided architecture information (a discovery
   report, ADRs, or what the user gives you) — never inferred from code
   yourself, that's `architecture-discovery`'s job:
   - people and external systems
   - the system boundary
   - (for Container) the containers and their communication paths
     (protocol, sync/async)
   - (for Dynamic) the scenario whose runtime collaboration is being shown
   - (for Deployment) the infrastructure nodes containers map to
   If the information given is insufficient, ask before drawing — don't
   invent containers or relationships.
3. Render as **Mermaid** (`C4Context` / `flowchart`) or **Structurizr DSL**;
   default to Mermaid for portability. Keep diagrams readable — split rather
   than overcrowd a single diagram.
4. Validate against `references/review-checklist.md`'s C4-specific criteria —
   the single review bar for every artifact type in this plugin.
   `references/c4-guidelines.md` holds the conventions and Mermaid examples
   those criteria check against. If the open question is whether the diagram
   still matches the *real system*, that is outside this skill's scope:
   request an `architecture-discovery` pass rather than reading source code
   yourself.

## Inputs

- Prefer an `architecture-discovery` report if one exists.
- Otherwise ask for: system purpose, known boundaries, containers (if known),
  data stores, and external dependencies. Don't proceed on assumptions where
  the answer materially changes the diagram — prefer asking for clarification
  over creating speculative architecture.

See `references/c4-guidelines.md` for conventions and examples.
