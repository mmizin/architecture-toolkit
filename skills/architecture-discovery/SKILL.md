---
name: architecture-discovery
description: >-
  Scan an existing codebase to reconstruct its architecture and surface the
  documentation it is missing. Detects languages, frameworks, modules,
  services, data stores, and integration points; infers component boundaries
  and data flows; and identifies significant architectural decisions that
  should be captured as ADRs. Use when the user wants to understand an
  unfamiliar project, bootstrap docs for a project that has none, or find
  undocumented architecture. Produces a discovery report that feeds the
  adr-expert, c4-expert, and arc42-expert skills.
---

# Architecture Discovery

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

Reconstruct what a project's architecture *actually is* and expose the gap
between reality and its documentation.

## When to use

- Onboarding onto an unfamiliar codebase.
- A project has little or no architecture documentation and needs a starting
  point ("scan and create the docs that should exist").
- You suspect significant decisions were made but never recorded.

## Workflow

1. **Inventory.** Detect languages, package managers, frameworks, build tools,
   containers, CI, and infra config. Read manifests and lockfiles first
   (`package.json`, `pyproject.toml`, `go.mod`, `pom.xml`, `Dockerfile`,
   `docker-compose.yml`, IaC files).
2. **Map structure.** Identify top-level modules/services, their
   responsibilities, and dependency direction. Note data stores, queues,
   external APIs, and auth boundaries.
3. **Infer decisions.** Flag architecturally significant choices evident from
   the code (framework selection, persistence strategy, sync vs async,
   monolith vs services, auth model) that lack an ADR.
4. **Assess doc gaps.** Compare what exists (`README`, `ARCHITECTURE.md`,
   `docs/`, ADRs, diagrams) against what the architecture warrants.
5. **Report.** Emit a discovery report:
   - System overview (one paragraph)
   - Component/module inventory with responsibilities
   - Inferred data flows and integration points
   - Candidate ADRs (decisions worth recording), ranked by significance
   - Documentation gaps and a recommended generation order

## Handing off

- Candidate decisions → `adr-expert`
- Component/container structure → `c4-expert`
- Full architecture write-up → `arc42-expert` / `architecture-documentation`

Do not invent architecture. Distinguish clearly between what is **observed in
the code** and what is **inferred** — mark inferences as such and verify with
the user.
