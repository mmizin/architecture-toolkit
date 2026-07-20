---
name: arc42-expert
description: >-
  Generate, update, and validate architecture documentation following the
  arc42 template (its 12 standard sections). Use when the user wants a
  structured architecture document for a system, wants to fill or refresh
  specific arc42 sections, or wants to check existing docs for completeness.
  Integrates ADRs (section 9) and C4 diagrams (sections 3, 5, 6) by reference
  rather than duplicating them.
---

# arc42 Expert

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

Produce and maintain architecture documentation using the arc42 structure — a
proven 12-section template that scales from a one-pager to full documentation.

## When to use

- Creating architecture documentation for a system from scratch.
- Filling or updating specific sections (e.g. quality goals, building blocks).
- Validating existing documentation for completeness and consistency.

## The 12 sections

1. Introduction & Goals
2. Constraints
3. Context & Scope  *(→ C4 System Context)*
4. Solution Strategy
5. Building Block View  *(→ C4 Container/Component)*
6. Runtime View  *(→ sequence / C4 dynamic)*
7. Deployment View
8. Cross-cutting Concepts
9. Architecture Decisions  *(→ link ADRs from adr-expert)*
10. Quality Requirements
11. Risks & Technical Debt
12. Glossary

## Workflow

1. Determine scope — full document vs specific sections. Don't pad empty
   sections; omit or mark "not applicable" honestly.
2. Pull existing material: discovery report, ADRs, C4 diagrams, READMEs.
3. Draft/update sections using `references/arc42-template.md`.
4. **Reference, don't duplicate**: section 9 links to ADRs; sections 3/5/6
   embed or link C4 diagrams.
5. Validate: are quality goals concrete and measurable? Is every building block
   traceable to a responsibility? Are risks and debt named honestly?

See `references/arc42-template.md` for the section prompts.
