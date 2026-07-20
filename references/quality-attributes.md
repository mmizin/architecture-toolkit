# Quality Attributes — Shared Vocabulary

The common set of quality attributes (a.k.a. non-functional requirements) used
consistently across ADRs, arc42 §10, and architecture reviews. Use these names
so decisions, quality scenarios, and reviews all speak the same language.

- **Performance** — response time, throughput, resource usage under load.
- **Scalability** — behavior as load, data volume, or users grow.
- **Availability** — uptime, fault tolerance, recovery from failure.
- **Reliability** — correctness of results over time, consistency guarantees.
- **Security** — confidentiality, integrity, authN/authZ, attack surface.
- **Maintainability** — ease of understanding, changing, and testing the code.
- **Simplicity** — cognitive load, number of moving parts, operational surface.
- **Testability** — ease of verifying correctness in isolation and in CI.
- **Deployability / Operability** — release process, rollback, observability,
  on-call burden.
- **Interoperability** — ease of integrating with other systems/teams.
- **Portability** — independence from a specific vendor, runtime, or platform.
- **Cost** — infrastructure, licensing, and engineering-time cost, now and at
  scale.
- **Developer Experience** — friction for engineers building on top of the
  decision.

## Usage

- ADRs: cite these under **Decision Drivers** (`references/adr-template.md`)
  instead of inventing ad hoc terms.
- arc42: build the quality tree in §10 from this list; only include attributes
  that actually matter for the system.
- Reviews: `architecture-reviewer` checks that trade-offs are stated in terms
  of these attributes, not vague adjectives ("better", "cleaner").
