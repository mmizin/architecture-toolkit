---
name: arc42-expert
description: >-
  Generate, update, and validate architecture documentation following the
  arc42 template (its 12 standard sections). Use when the user wants a
  structured architecture document for a system, wants to fill or refresh
  specific arc42 sections, or wants to check existing arc42 documentation for
  structural completeness. Integrates ADRs (section 9) and C4 diagrams
  (sections 3, 5, 6, 7) by reference rather than duplicating them.
---

# arc42 Expert

> Status: v0.1 stub — structure and workflow are defined; deepen during use.

Produce and maintain architecture documentation using the arc42 structure — a
proven 12-section template that scales from a one-pager to full documentation.

## Scope

**Responsible for:**
- creating and maintaining arc42 architecture documentation
- structuring architecture knowledge across the 12 arc42 sections
- connecting arc42 sections to the relevant ADRs and C4 diagrams by reference
- validating an arc42 document's own structural completeness

**Not responsible for:**
- discovering architecture from source code
- making or rewriting architectural decisions
- judging whether the underlying architecture or its decisions are good
- repository-wide documentation consistency audits

**Escalates to:**
- `architecture-discovery` — when architecture knowledge is missing rather
  than just undocumented; don't infer it from the code yourself.
- `adr-expert` — when §9 needs to cite a decision that has no ADR yet.
- `architecture-reviewer` — when the question is architectural quality, not
  documentation completeness.
- `architecture-librarian` — when resolving a consistency question requires
  comparing this document with other repository artifacts.

## When to use

- Creating architecture documentation for a system from scratch.
- Filling or updating specific sections (e.g. quality goals, building blocks).
- Validating an existing arc42 document for structural completeness.

## The 12 sections

1. Introduction & Goals
2. Constraints
3. Context & Scope  *(→ C4 System Context)*
4. Solution Strategy
5. Building Block View  *(→ C4 Container/Component)*
6. Runtime View  *(→ sequence / C4 Dynamic)*
7. Deployment View  *(→ C4 Deployment)*
8. Cross-cutting Concepts
9. Architecture Decisions  *(→ reference ADRs; do not restate their
   reasoning)*
10. Quality Requirements  *(use the shared vocabulary in
    `references/quality-attributes.md`; express important qualities as
    concrete quality scenarios, not vague adjectives)*
11. Risks & Technical Debt
12. Glossary

## Workflow

See `references/terminology.md` for artifact ownership conventions: arc42
narrates and links, it doesn't own decision rationale (ADRs) or structural
views (C4).

1. Determine scope — full document vs specific sections. Don't pad empty
   sections; omit or mark "not applicable" honestly.
2. Gather available inputs: an existing discovery report, ADRs, C4 diagrams,
   READMEs. If architecture knowledge is missing rather than just
   undocumented, hand off to `architecture-discovery` instead of inferring it
   yourself.
3. Draft/update sections using `references/arc42-template.md`.
4. **Reference, don't duplicate**: section 9 links to ADRs; sections 3/5/6/7
   embed or link C4 diagrams.
5. Validate against `references/review-checklist.md`. Apply only the
   arc42-specific criteria and the documentation-completeness dimensions;
   do not evaluate architectural correctness — that's
   `architecture-reviewer`.

See `references/arc42-template.md` for the section prompts.
