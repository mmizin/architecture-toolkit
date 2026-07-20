---
name: c4-expert
description: >-
  Create and review C4 model diagrams (System Context, Container, Component,
  and optionally Code) for a software system. Produces diagrams as text
  (Mermaid or Structurizr DSL) that live alongside the code. Use when the user
  wants to visualize system architecture, document containers and their
  interactions, or keep diagrams in sync with the real system. Works from an
  architecture-discovery report or from a description the user provides.
---

# C4 Expert

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

Model software architecture with the C4 approach: a small hierarchy of diagrams
at increasing zoom levels, each with a clear audience.

## When to use

- Documenting a system's context and container structure.
- Visualizing how services, data stores, and external systems interact.
- Reviewing an existing diagram for correctness and appropriate abstraction.

## The four levels (use only what the audience needs)

1. **System Context** — the system as one box, its users, and neighbouring
   systems. For everyone.
2. **Container** — deployable/runnable units (apps, services, databases) and
   their interactions. For technical stakeholders. *Usually the most valuable.*
3. **Component** — components inside one container. Only where it earns its keep.
4. **Code** — class/schema level. Rarely worth maintaining by hand.

## Workflow

1. Establish scope and audience — which level(s) are actually needed.
2. Identify people, the system in scope, external systems, and (for Container)
   the containers and their communication (protocol, sync/async).
3. Render as **Mermaid** (`C4Context` / `flowchart`) or **Structurizr DSL**;
   default to Mermaid for portability.
4. Review against `references/c4-guidelines.md`: right abstraction level, every
   relationship labelled with intent + technology, no mixed levels in one
   diagram.

## Inputs

- Prefer an `architecture-discovery` report if one exists.
- Otherwise ask for: system purpose, main containers, data stores, and external
  dependencies.

See `references/c4-guidelines.md` for conventions and examples.
