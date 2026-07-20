---
name: architecture-bootstrap
description: >-
  Entry point for a project with no (or stale) architecture documentation.
  Given a fresh codebase, sequence architecture-discovery, adr-expert,
  c4-expert, arc42-expert, and architecture-librarian into one coherent
  documentation set — README/ARCHITECTURE overview, arc42 sections, C4
  diagrams, and an ADR index — with consistent cross-links. Use when the
  user wants to "document the whole project" or "create the docs a project
  is missing" rather than a single artifact, and doesn't know which specific
  skill to start with.
---

# Architecture Bootstrap

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

The entry point for "document this project from zero" and "backfill the docs
a project should have." It doesn't produce a single artifact itself — it
determines what documentation the project needs and guides the sequence of
skills that produce it.

## Scope

**Responsible for:**
- deciding the documentation starting point for an undocumented project
- sequencing architecture documentation work across the other modules
- proposing the initial documentation plan

**Not responsible for:**
- discovering architecture from code (that's `architecture-discovery`)
- writing ADRs, C4 diagrams, or arc42 sections itself
- maintaining documentation consistency over time

**Escalates to:**
- `architecture-discovery` — to ground the plan in the real system.
- `adr-expert`, `c4-expert`, `arc42-expert` — to produce each artifact.
- `architecture-librarian` — to keep the resulting set healthy afterward.

## When to use

- A project has no (or stale) architecture documentation and the user wants a
  complete, consistent baseline.
- The user asks to "scan and create the documentation" without naming a
  specific format.
- The user doesn't know whether they need an ADR, a C4 diagram, or arc42 —
  they just know they have nothing.

## Workflow

1. **Discover.** Run `architecture-discovery` first (or ingest its report) to
   ground everything in the real system.
2. **Propose a plan.** Present the intended documentation set and order, e.g.:
   - `ARCHITECTURE.md` overview (context + solution strategy)
   - C4 System Context + Container diagrams
   - arc42 sections that the system warrants
   - ADRs for the significant decisions discovery surfaced
   - an initial ADR index (`docs/adr/README.md`), if ADRs are created
   Get confirmation before generating in bulk.
3. **Apply the appropriate skills** — `arc42-expert`, `c4-expert`,
   `adr-expert` — one artifact at a time, following each module's own
   conventions.
4. **Connect the artifacts.** Ensure the diagrams, arc42 sections, and ADRs
   just created reference each other correctly and don't duplicate content.
   Flag anything inferred rather than observed for the user to confirm.
5. **Hand off for ongoing health.** Recommend an `architecture-librarian`
   pass to verify repository-wide consistency and index health going
   forward — this skill sets up the initial map, it doesn't maintain it.

## Note on sequencing, not orchestration

Claude Code skills can't call each other programmatically — this skill does
not invoke the others at runtime. It encodes the **plan and the order**: it
tells the user (or the agent driving the session) which skill to apply next
and why, and keeps the user in the loop between major artifacts. The actual
generation of each artifact happens by applying that skill's own guidance
directly in the session.
