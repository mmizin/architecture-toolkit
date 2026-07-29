---
name: architecture-discovery
description: >-
  Reconstructs architecture from repository evidence: technology landscape,
  component boundaries, integrations, data flows, candidate architectural
  decisions that may require ADRs, and evidence-backed Implementation
  Findings (observed implementation conditions that carry a plausible risk,
  not recommendations). Use when the user
  wants to understand an unfamiliar project, bootstrap docs for a project
  that has none, or find undocumented architecture. Produces a discovery
  report that feeds the adr-expert, c4-expert, and arc42-expert skills.
---

# Architecture Discovery

> Status: Initial implementation — structure and workflow are defined; deepen
> during use.

Reconstruct what a project's architecture *actually is* and expose the gap
between reality and its documentation.

## Scope

**Responsible for:**
- reconstructing the current architecture from observable evidence in the
  repository
- identifying components, integrations, and boundaries
- surfacing candidate architectural decisions and documentation gaps
- recording Implementation Findings — evidence-backed, observed
  implementation conditions that carry a plausible risk, distinct from a
  candidate decision or a missing document (see
  `../../references/FINDINGS-CONTRACT.md`)

**Not responsible for:**
- judging whether the discovered architecture is good
  (`architecture-reviewer`'s job)
- creating ADRs, C4 diagrams, or arc42 sections itself
- designing a target architecture
- ordering or sequencing the documentation work implied by the gaps it finds
  (that's `architecture-bootstrap`)
- auditing consistency across existing documentation artifacts
  (`architecture-librarian`'s job)
- recommending fixes, deciding severity, or changing an Implementation
  Finding's status once filed — discovery only creates findings and reports
  on a rescan whether their evidence still holds (see
  `../../references/FINDINGS-CONTRACT.md` R-003, R-004, R-007)

**Escalates to:**
- `adr-expert` — to turn a candidate decision into an actual ADR, or, when a
  finding's resolution warrants one, to record that resolution as an ADR.
  Writing the ADR is not itself what closes the finding — the finding's
  `Status`/`Resolved by:` update follows separately, per
  `../../references/FINDINGS-CONTRACT.md` R-007.
- `c4-expert` — to diagram the discovered boundaries.
- `arc42-expert` — to write up the full architecture narrative.
- `architecture-reviewer` — when architecture quality needs evaluation.
- `architecture-librarian` — when existing documentation needs a
  consistency audit rather than a fresh scan.
- `architecture-bootstrap` — when this is the first step of documenting a
  project from zero, not a one-off scan.

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
   external APIs, and auth boundaries. Avoid assuming runtime/service
   boundaries from folder structure alone — `src/payment/` and `src/user/`
   may be two packages in one deployable, not two services.
3. **Feature-scoped verification checklist (conditional).** When discovery
   is scoped to a single feature, endpoint, or component — rather than a
   whole-repository bootstrap — run this checklist before writing the
   report. Each item is a concrete search, not a judgment call — skipping
   one because "it's probably fine" is exactly the failure mode this
   checklist exists to close. This step exists to ensure feature-scoped
   discovery establishes readily verifiable evidence — reachability, ADR
   applicability and revision, implementation status — before producing
   the report, rather than leaving it for a downstream consumer to
   discover independently.
   1. **Frontend callers.** Search the frontend/client tree for any
      reference to the endpoint/route/function under discovery (route
      string, RPC method name, generated client method). Report a positive
      or a negative finding explicitly — "no caller found" is itself
      evidence, not an absence of evidence, and belongs in the report.
   2. **Test callers.** Same search across test suites and scripts (not
      just the primary frontend). Report a positive or negative finding
      explicitly, same as (1).
   3. **Historical callers.** Only run this when documentation (README,
      architecture docs) describes a caller that is absent from the
      current tree — check git history for a removed caller. This is a
      targeted trigger, not a default step: git-history search is
      expensive and unnecessary when nothing in the docs claims a caller
      exists. A stale doc describing removed infrastructure is a distinct,
      reportable finding from "never had a caller."
   4. **Related ADRs.** Search the ADR directory for any decision whose
      Context or Decision section names this feature's files, functions,
      or line numbers — not just ADRs whose title looks topically related.
      Search by filenames, symbols, routes, DTOs, services, repositories,
      and line references, not just topical keywords. Accepted ADRs
      frequently reach into implementation detail (exact write sites,
      exact line numbers) that only a text search surfaces.
   5. **Accepted vs. superseded revision.** For every ADR found in (4),
      read its actual current Decision section — never infer content from
      the Context, Options, or an early Revision's prose. Multi-revision
      ADRs are common and early revisions are explicitly superseded;
      citing revision N's Context section to describe revision M's
      Decision is the single highest-cost error this checklist targets.
      Never cite an ADR revision until the Accepted revision has been
      explicitly identified.
   6. **Implementation status of each Accepted ADR.** An "Accepted" status
      means the decision is settled, not that it shipped. Verify directly
      against source (grep for the mechanism named — a throttle guard, an
      exception type, a column type) and classify as one of: **Accepted +
      Implemented**, **Accepted + Partially implemented**, or **Accepted +
      Not implemented**. State this status explicitly and uniformly across
      every ADR cited — don't apply "not yet implemented" hedging to one
      ADR and omit it for another out of inconsistent scrutiny.
   7. **Existing DISCOVERY.md.** Check whether a Discovery artifact already
      exists for this feature before producing a new one. If one exists,
      this is an update/rescan, not a fresh Create.
   8. **Existing feature documentation.** Search existing feature-level
      documentation artifacts (for example `docs/features/*` or
      equivalent locations) for any document that already establishes
      evidence this discovery would otherwise re-derive — shared field
      sets, shared write paths, shared actors. Reuse cited evidence rather
      than re-deriving it, but verify the citation still matches current
      source before reusing it (a sibling artifact can itself be stale).
   9. **Reachability / retirement status.** For any endpoint, route, or
      entry point under discovery, explicitly answer: is it currently
      reachable by any caller in the repository? No caller found is not
      the same claim as a dead endpoint — don't collapse the two. If
      evidence suggests no caller — don't assume intent (deliberate
      API-only surface, planned future caller, or retirement candidate) —
      report the absence of a caller as a fact and record the intent
      question as unresolved for the appropriate downstream artifact owner
      to decide, not for discovery to guess at.
   10. **Reused evidence verification.** For every piece of evidence reused
       from a sibling documentation artifact, an ADR, or a prior Discovery
       doc (items 4-8), re-verify it against current source before citing
       it in this report. Do not inherit a prior document's conclusion on
       the assumption that it still holds — a stale referenced artifact
       can cause incorrect evidence to enter this report, which this check
       prevents. If re-verification fails to confirm a
       reused statement, treat it as stale until proven otherwise — do not
       carry the original conclusion forward on the assumption that
       verification was inconclusive rather than contradictory. This is
       evidence validation for reused facts feeding this report, not a
       general documentation-consistency audit — that remains
       `architecture-librarian`'s job.

   Report each checklist item's outcome explicitly in the discovery
   report, even when the answer is "none found" — an unchecked item is
   indistinguishable from a checked item with no findings unless the
   report says which one happened. This step does not apply to a
   whole-repository bootstrap scan (see `architecture-bootstrap`) — a
   full-repo scan doesn't need "does this endpoint have a frontend caller"
   as a checklist item, but a single-feature scan always does.
4. **Identify candidate decisions.** Surface architectural choices visible in
   the repository that may represent intentional decisions (framework
   selection, persistence strategy, sync vs async, monolith vs services,
   auth model) and lack an ADR. The code only shows what was done, not why —
   don't assume the rationale (a library import isn't proof of a deliberate
   choice; it could be a leftover experiment or an accident). Do not promote
   every dependency into a candidate decision — using a library (e.g.
   Jackson) isn't architectural; choosing an architectural style (e.g.
   event-driven) is. Mark these as **candidate ADRs** (see
   `../../references/terminology.md`) requiring confirmation, not settled
   decisions.
5. **Assess doc gaps.** Compare what exists (`README`, `ARCHITECTURE.md`,
   `docs/`, ADRs, diagrams) against what the discovered architecture
   warrants — obvious coverage gaps only. Whether the existing documentation
   is internally consistent with itself is `architecture-librarian`'s check,
   not this one.
6. **Record Implementation Findings, only if any exist.** While mapping
   structure and gaps, note any code behavior that carries a plausible,
   evidence-backed risk and that isn't already a candidate decision or a doc
   gap — apply the R-005 filing checks in
   `../../references/FINDINGS-CONTRACT.md` before filing. `FINDINGS.md` is a
   conditional artifact: create or update it only when at least one finding
   is filed; a scan that surfaces none produces no `FINDINGS.md` and this
   step is a no-op. Write each finding as its own entry per
   `../../references/findings-template.md`: evidence and impact only, no fix
   recommendation, no severity score. On a rescan of a project with an
   existing `FINDINGS.md`, never rewrite an existing entry in place —
   discovery only records whether an entry's original evidence still holds.
   For `Open` entries, discovery only reports whether the evidence still
   holds; it does not edit the entry. For `Closed`/`Accepted` entries this is
   additionally forbidden by the lifecycle rules — a regression against
   either is filed as a new entry with `Related: Supersedes IF-0NN`
   (`../../references/FINDINGS-CONTRACT.md` R-007).
7. **Report.** Emit a discovery report:
   - System overview (one paragraph)
   - Component/module inventory with responsibilities
   - Observed data flows and integration points, and inferred ones, kept
     separate — every inference stated with the evidence that led to it
   - Candidate ADRs (potential decisions requiring confirmation), ranked by
     significance
   - Documentation gaps — an unordered inventory of what is missing relative
     to what the discovered architecture warrants. Report the evidence, not a
     plan: sequencing the work is `architecture-bootstrap`'s job.
   - Implementation Findings — if step 6 filed any, a pointer to
     `FINDINGS.md` plus the list of IDs touched this scan (new and
     rescanned). The report never inlines finding content — `FINDINGS.md` is
     the source of record, not the discovery report. Omit this bullet
     entirely when no findings were filed.

## Handing off

- Candidate decisions → `adr-expert`
- Component/container structure → `c4-expert`
- Full architecture write-up → `arc42-expert`
- Documenting a project from zero, start to finish → `architecture-bootstrap`
- A feature where checklist item 9 (reachability) resolves to "no current
  caller" → flag this as a reachability uncertainty for downstream
  consumers, since it can affect downstream artifact scope and
  assumptions.
- Implementation Findings needing a decision → `adr-expert`, to record the
  resolution as an ADR that the `FINDINGS.md` entry can then cite via
  `Resolved by:`. Findings that don't need an ADR are resolved through an
  external owner workflow (a code change, an issue tracker, a direct risk
  acceptance) — discovery does not set `Accepted` or
  `Closed` itself in any case, ADR-backed or not
  (`../../references/FINDINGS-CONTRACT.md` R-007).

Do not invent architecture. Distinguish clearly between what is **observed in
the code** and what is **inferred** — mark inferences as such, state the
evidence behind every inference, and verify with the user.

The same discipline applies to Implementation Findings: report what was
observed and the evidence for it, never what should be done about it —
`../../references/FINDINGS-CONTRACT.md` governs the rules, and any drift
from them (a recommendation, a severity score, a status change) is a
conformance violation in this skill's output, not a judgment call.
