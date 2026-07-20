---
name: c4-expert
description: >-
  Create and review C4 model diagrams (System Context, Container, Component,
  and optionally Code) for a software system. Produces diagrams as text
  (Mermaid or Structurizr DSL) that live alongside the code. Use when the user
  wants to visualize system architecture, document containers and their
  interactions, or keep diagrams consistent with documented system
  boundaries. Works from an architecture-discovery report or from a
  description the user provides.
---

# C4 Expert

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

Model software architecture with the C4 approach: a small hierarchy of diagrams
at increasing zoom levels, each with a clear audience.

## Scope

**Responsible for:**
- creating and maintaining C4 diagrams
- choosing the appropriate C4 abstraction level for the audience
- documenting system context, containers, and components visually
- validating a diagram's own structure against C4 conventions

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
3. **Component** — components inside one container. Only where it earns its keep.
4. **Code** — class/schema level. Rarely worth maintaining by hand.

Diagrams show structure and relationships only — see
`references/terminology.md`'s "Artifact ownership" entry for what C4 owns
versus what belongs to an ADR or arc42.

## Workflow

1. Establish scope and audience — which level(s) are actually needed.
2. Identify people, the system in scope, external systems, and (for Container)
   the containers and their communication (protocol, sync/async) from
   available architecture information — a discovery report, ADRs, or what
   the user provides. Don't infer unknown boundaries from code yourself;
   that's `architecture-discovery`'s job. If the information given is
   insufficient, ask before drawing — don't invent containers or
   relationships.
3. Render as **Mermaid** (`C4Context` / `flowchart`) or **Structurizr DSL**;
   default to Mermaid for portability.
4. Validate against `references/review-checklist.md`'s C4-specific
   criteria, which point to `references/c4-guidelines.md` for the diagram
   conventions themselves (abstraction level, labelled relationships, no
   mixed levels).

## Inputs

- Prefer an `architecture-discovery` report if one exists.
- Otherwise ask for: system purpose, main containers, data stores, and external
  dependencies. Don't proceed on assumptions where the answer materially
  changes the diagram.

See `references/c4-guidelines.md` for conventions and examples.
