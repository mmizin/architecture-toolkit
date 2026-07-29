# Draft: add a "Feature-scoped verification checklist" step to architecture-discovery

Rationale: across Feature 9/10 in CryptoSandboxQA, `product-reviewer` had to
independently discover facts a Discovery pass should have surfaced first —
reachability (no frontend/test caller), applicable ADRs and their exact
Accepted-vs-superseded revision, and implementation status of an "Accepted"
decision. This turned review rounds into de facto Discovery, which is more
expensive per finding than catching it upfront. The fix is a mandatory
checklist specifically for the common case of "run Discovery for one
feature/endpoint," not the whole-repo bootstrap case the skill currently
optimizes prose around.

The key lesson from Feature 9 and Feature 10 is that `product-reviewer`
repeatedly surfaced missing evidence, not document-quality defects. The
checklist below addresses that specific failure mode while keeping the
existing Discovery workflow intact — it is not a redesign of Discovery's
scope or output model.

---

## Proposed insertion point

New step between current step 2 ("Map structure") and step 3 ("Identify
candidate decisions"), OR as an explicit sub-mode when discovery scope is a
single feature/endpoint rather than a whole repository. Given the skill's
existing workflow is bootstrap-oriented (whole-repo), I'd frame it as a
**scope-conditional checklist**, not a universal added step — a full-repo
scan doesn't need "does this endpoint have a frontend caller" as a checklist
item, but a single-feature scan always does.

## Proposed new section

```markdown
## Feature-scoped verification checklist

When discovery is scoped to a single feature, endpoint, or component
(rather than a whole-repository bootstrap), run this checklist before
writing the report. Each item is a concrete search, not a judgment call —
skipping one because "it's probably fine" is exactly the failure mode this
checklist exists to close.

1. **Frontend callers.** Search the frontend/client tree for any reference
   to the endpoint/route/function under discovery (route string, RPC
   method name, generated client method). Report a positive or a negative
   finding explicitly — "no caller found" is itself evidence, not an
   absence of evidence, and belongs in the report.
2. **Test callers.** Same search across test suites and scripts (not just
   the primary frontend). Report a positive or negative finding
   explicitly, same as (1).
3. **Historical callers.** Only run this when documentation (README,
   architecture docs) describes a caller that is absent from the current
   tree — check git history for a removed caller. This is a targeted
   trigger, not a default step: git-history search is expensive and
   unnecessary when nothing in the docs claims a caller exists. A stale
   doc describing removed infrastructure is a distinct, reportable finding
   from "never had a caller."
4. **Related ADRs.** Search the ADR directory for any decision whose
   Context or Decision section names this feature's files, functions, or
   line numbers — not just ADRs whose title looks topically related.
   Search by filenames, symbols, routes, DTOs, services, repositories, and
   line references, not just topical keywords. Accepted ADRs frequently
   reach into implementation detail (exact write sites, exact line
   numbers) that only a text search surfaces.
5. **Accepted vs. superseded revision.** For every ADR found in (4), read
   its actual current Decision section — never infer content from the
   Context, Options, or an early Revision's prose. Multi-revision ADRs are
   common in this project style and early revisions are explicitly
   superseded; citing revision N's Context section to describe revision
   M's Decision is the single highest-cost error this checklist targets.
   Never cite an ADR revision until the Accepted revision has been
   explicitly identified.
6. **Implementation status of each Accepted ADR.** An "Accepted" status
   means the decision is settled, not that it shipped. Verify directly
   against source (grep for the mechanism named — a throttle guard, an
   exception type, a column type) and classify as one of: **Accepted +
   Implemented**, **Accepted + Partially implemented**, or **Accepted +
   Not implemented**. State this status explicitly and uniformly across
   every ADR cited — don't apply "not yet implemented" hedging to one ADR
   and omit it for another out of inconsistent scrutiny.
7. **Existing DISCOVERY.md.** Check whether a Discovery artifact already
   exists for this feature before producing a new one. If one exists,
   this is an update/rescan, not a fresh Create.
8. **Sibling PRDs.** Search `docs/features/*/PRD.md` (or equivalent) for
   any document that already establishes evidence this discovery would
   otherwise re-derive — shared field sets, shared write paths, shared
   actors. Reuse cited evidence rather than re-deriving it, but verify the
   citation still matches current source before reusing it (a sibling PRD
   can itself be stale).
9. **Reachability / retirement status.** For any endpoint, route, or
   entry point under discovery, explicitly answer: is it currently
   reachable by any caller in the repository? No caller found is not the
   same claim as a dead endpoint — don't collapse the two. If evidence
   suggests no caller — don't assume intent (deliberate API-only surface,
   planned future caller, or retirement candidate) — report the absence of
   a caller as a fact and record the intent question as unresolved for the
   PRD/ADR layer to decide, not for discovery to guess at.
10. **Cross-document consistency.** For every piece of evidence reused
    from a sibling PRD, an ADR, or a prior Discovery doc (items 4-8),
    re-verify it against current source before citing it in this report.
    Do not inherit a prior document's conclusion on the assumption that it
    still holds — a sibling document's staleness is exactly what produced
    the highest-cost errors this checklist targets (items 5-6 above). If
    re-verification fails to confirm a reused statement, treat it as stale
    until proven otherwise — do not carry the original conclusion forward
    on the assumption that verification was inconclusive rather than
    contradictory.

Report each checklist item's outcome explicitly in the discovery report,
even when the answer is "none found" — an unchecked item is
indistinguishable from a checked item with no findings unless the report
says which one happened.
```

## Handoff note to add

Add one line under "Handing off":

> - A feature where checklist item 9 (reachability) resolves to "no
>   current caller" → flag for the consuming PRD to record this as its
>   highest-priority Open Question, since it typically determines the
>   shape of the rest of the document.

---

## Why scope-conditional rather than universal

The skill's current workflow (steps 1-6) is written for "reconstruct an
unfamiliar project's whole architecture." Items like "frontend caller
search" don't make sense as a repo-wide step (you'd be checking every
route against every possible caller) — they're specific to the common
sub-case this project actually uses architecture-discovery for: verifying
one feature/PRD's evidence before or after a product-reviewer round. Worth
naming that distinction explicitly in the skill rather than bolting a
single-feature checklist onto whole-repo prose.

## Explicitly out of scope for this proposal

Two related ideas came up during review and are deliberately excluded here
to avoid delaying adoption of this validated process improvement:

- **Confidence calibration** (tagging every fact as Confirmed / Inferred /
  Reused evidence / Open Question) — a genuinely useful idea, but a change
  to Discovery's reporting model, not a checklist item.
- **Discovery Modes** (Repository / Feature / Regression / Evidence
  Refresh as named, distinct modes) — the underlying distinction this
  draft already makes scope-conditional, but formalizing it as named modes
  is a larger restructuring of the skill.

Both should be raised as a separate follow-up proposal once this narrower
fix has been adopted and validated.

## Success criteria

This proposal is considered successful if, across subsequent feature-level
Discovery/PRD/Review cycles:

- `product-reviewer` no longer discovers missing evidence (reachability,
  ADR applicability, ADR revision, implementation status) that should have
  been established during Discovery.
- Feature-level Discovery consistently resolves reachability, ADR
  applicability, and implementation status before PRD drafting begins.
- Review rounds shift toward document-quality findings and away from
  evidence-collection findings.
