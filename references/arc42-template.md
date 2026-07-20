# arc42 Architecture Documentation — <System Name>

> The arc42 template, 12 sections. Fill what the system warrants; mark sections
> "not applicable" honestly rather than padding them. Reference ADRs and C4
> diagrams instead of duplicating their content.

## 1. Introduction and Goals
Requirements overview, top 3–5 quality goals, key stakeholders.

## 2. Architecture Constraints
Technical, organizational, and conventions the architecture must obey.

## 3. Context and Scope
Business and technical context — who/what the system talks to.
→ C4 **System Context** diagram.

## 4. Solution Strategy
The fundamental decisions and approaches: technology, top-level decomposition,
how quality goals are achieved. Summarize; link ADRs for the reasoning.

## 5. Building Block View
Static decomposition into building blocks and their responsibilities.
→ C4 **Container / Component** diagrams.

## 6. Runtime View
How building blocks collaborate for key scenarios.
→ sequence / C4 **Dynamic** diagrams.

## 7. Deployment View
Infrastructure, environments, and mapping of building blocks to nodes.
→ C4 **Deployment** diagrams.

## 8. Cross-cutting Concepts
Recurring patterns and concepts: persistence, security, error handling,
logging, i18n, etc.

## 9. Architecture Decisions
Important decisions with rationale.
→ Link the ADRs (see `references/adr-template.md`); do not restate them here.
Do not summarize accepted ADRs unless needed for orientation.

## 10. Quality Requirements
Quality tree and concrete, testable quality scenarios.

## 11. Risks and Technical Debt
Known risks and debt, named honestly, with mitigation where known.

## 12. Glossary
Domain and technical terms, defined once.