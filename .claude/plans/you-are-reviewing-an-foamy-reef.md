# Architecture Toolkit — Incremental Improvement Plan

## Context

This plugin is a documentation-architecture toolkit: 5 skills, 2 agents, 9 shared
references, ~1,150 lines of Markdown, no code and no tests. An external review
(`review.md`) produced 19 findings. I verified all of them against the current
tree and added two more.

**The diagnosis is not that the architecture is wrong.** All 7 modules carry a
complete `Responsible for` / `Not responsible for` / `Escalates to` block; all 9
`references/*.md` paths resolve; there are zero broken links; there is no god
skill and no hidden orchestration. The uniformity is the plugin's biggest asset.

The actual problem is narrower and has a single shape:

> The module layer is governed by a template every `SKILL.md` follows. The
> reference layer and the README have no template, no owner, and no review — so
> that is where every defect lives.

Evidence: the README has **1** commit (the initial one) while modules have 2–4.
Every boundary violation found is either inside a reference file, inside the
README, or at `architecture-bootstrap/SKILL.md:56` — the one place a module
describes an artifact (`ARCHITECTURE.md`) that no reference file defines.

**Intended outcome:** repair the drift, formalize ownership of the reference
layer, and preserve the module boundaries exactly as they are. No restructuring.

### Scope confirmed with the user

- Execute **Phases 0–3**. Phases 4–5 are documented here but deferred.
- CI/lint automation: **documented in Phase 5, not built now.**
- F5 (C4 dynamic/deployment): **declare ownership only, author content later.**

### Verified baseline (independent of `review.md`)

| Check | Result |
|---|---|
| All 7 modules have all 3 boundary headings | ✅ pass |
| All 9 `references/*.md` paths resolve | ✅ pass (zero broken links) |
| Module names in repo that resolve to no module | ❌ `architecture-documentation` only |
| `architecture-bootstrap` mentions in README | ❌ **zero** |
| Modules missing a `> Status:` marker | ❌ `adr-expert` only |
| CI / test tooling present | none (`.github/` does not exist) |
| Working tree | clean except untracked `review.md` |

### Two findings the original review missed

- **F20** — `architecture-bootstrap` is the only module citing **zero** reference
  files, despite coordinating all the others. It never points at
  `terminology.md`, so the module most likely to introduce a new artifact type
  is the one least anchored to the shared glossary. This is the mechanism behind
  F3, not a coincidence.
- **F21** — `decision-drivers.md` is cited by two *reference* files but by **zero**
  modules. It is a second-order orphan: reachable on paper, unreachable in
  practice. `plugin-design-principles.md` (F9) is a first-order orphan with zero
  citations of any kind.

### One correction to the review's F5

`review.md` treats arc42 §6 and §7 as the same defect. They are not:

- **§6 Runtime View** is annotated `→ sequence / C4 dynamic diagrams`
  (`arc42-template.md:27`, `arc42-expert/SKILL.md:55`) — a **dangling pointer**
  to a diagram type `c4-expert` does not define.
- **§7 Deployment View** carries **no** annotation anywhere — a **silent gap**.

They need different fixes. §6 needs its target defined; §7 needs a decision about
whether it has a target at all.

---

## Improvement Items

Severity uses the plugin's own taxonomy (`references/severity-levels.md`).
Category uses the scheme requested for this plan.

---

### F1 — README documents a module that has never existed

**Problem.** README names `architecture-documentation` in three places
(`README.md:26` module table, `:50` directory layout, `:81` greenfield entry
point). No such directory exists in any commit. The shipped module is
`architecture-bootstrap`, which appears in the README **zero** times.

**Category.** Documentation drift
**Severity.** Critical

**Dependencies.** None — this is the root of the README cluster and must land
before F16, F18, and F9's README half, all of which edit the same file.

**Change direction.** Replace all three occurrences with `architecture-bootstrap`.
Then re-derive the `## Recommended workflow` section (`:64–78`) from the seven
modules' actual `Escalates to` blocks rather than editing it in place — it has
never been reconciled against the modules as shipped, so in-place editing would
preserve unverified claims.

**Risk.** Low mechanically. The real risk is *stopping at the rename*: the
workflow section was written before the modules existed in their current form,
so a correct name inside a stale workflow is a subtler defect than the current
obvious one. Re-derive, do not patch.

**Out-of-repo dependency.** This session's *additional working directory* is set
to `skills/architecture-documentation` — a path that does not exist on disk. The
stale name is live in environment config, not just the README, so it is
propagating rather than decaying. **The user must update this manually**; it is
not a file I can fix.

**Validation.**
- `grep -r "architecture-documentation" --include='*.md' .` returns hits only in `review.md`
- The module-name lint (below) reports `UNKNOWN` for nothing
- Every module named in the workflow section appears in the `Escalates to` block of the module preceding it

---

### F2 — Librarian's declared write behavior exceeds its tool grant

**Problem.** `agents/architecture-librarian.md:10` grants `tools: Read, Grep, Glob`.
The description (`:8–9`) says it "only edits index/cross-link files when
explicitly asked" and the closing paragraph (`:95–96`) says "Apply edits only to
index/cross-link files." The agent has no `Edit` or `Write` and cannot perform
the write it twice promises.

**Category.** Correctness issue (runtime capability mismatch)
**Severity.** High

**Dependencies.** None. Requires a direction decision, already made: **keep it
read-only** (option B).

**Change direction.** Leave the tool grant untouched. Reword both the description
and the closing paragraph to "proposes edits as a patch for the user or
`adr-expert` to apply." This matches `architecture-reviewer.md:78`
(`Do not edit the artifact.`) and the plugin's propose-don't-apply posture.

**Risk.** The opposite fix — granting `Edit` — is the tempting one and it is
wrong. It would make the librarian the only module that can silently mutate the
documentation set it audits, and the approval gate would become the sole
guardrail where currently the capability boundary enforces it. Do not grant Edit.

**Validation.**
- No sentence in the file asserts the agent applies, writes, or modifies anything
- Frontmatter `tools:` unchanged
- The proposed-patch wording names who applies the patch

---

### F3 — `ARCHITECTURE.md` is an unowned artifact carrying arc42 content

**Problem.** `architecture-bootstrap/SKILL.md:32` disclaims "writing ADRs, C4
diagrams, or arc42 sections itself." Its workflow step 2 (`:56`) then proposes
`ARCHITECTURE.md` overview **"(context + solution strategy)"** — which is arc42
§3 and §4 by name (`arc42-template.md:13,17`). `ARCHITECTURE.md` appears in
**zero** reference files; `terminology.md:33-40` assigns owners only to ADRs, C4,
and arc42.

**Category.** Ownership/boundary issue
**Severity.** High

**Dependencies.** **Ownership must be decided before the bootstrap workflow is
touched.** Decision made: **`architecture-bootstrap` owns `ARCHITECTURE.md` as an
entry point and navigation map — not as architecture content.** This is a third
path, better than either option in `review.md`.

Order within the item: (1) add the definition to `terminology.md`, (2) then edit
`bootstrap:56` to conform to it. Reversing this leaves the glossary and the skill
disagreeing — the exact drift class being repaired.

**Change direction.**
- `terminology.md` — extend the Artifact ownership section with `ARCHITECTURE.md`:
  purpose = entry point / navigation / links and one-line summaries; owner =
  `architecture-bootstrap`; explicitly **does not** replace arc42 sections, ADRs,
  or C4 diagrams.
- `bootstrap:56` — the parenthetical `(context + solution strategy)` **is** the
  violation. Replace with wording describing links and summaries, not content.
- Confirm `architecture-discovery/SKILL.md:72`, which reads `ARCHITECTURE.md` as
  a doc-gap input, remains consistent — under the navigation reading it does.

**Risk.** Two failure modes. (a) Adding the terminology entry without editing
`bootstrap:56` leaves the contradiction intact and adds a glossary entry that
lies. (b) Defining `ARCHITECTURE.md` too broadly re-creates the duplication
`terminology.md:33-40` warns against — if it may contain "an overview of the
architecture," §3/§4 exist in two places with no source of truth and the
librarian has no rule for which wins. Keep the definition narrow: links,
summaries, navigation.

**Validation.**
- `ARCHITECTURE.md` has exactly one named owner in `terminology.md`
- No line in `bootstrap` describes `ARCHITECTURE.md` as containing architecture content
- `grep -rn "ARCHITECTURE.md" skills/ agents/ references/` — every hit consistent with the navigation definition

---

### F4 — Discovery and Bootstrap both emit a generation order

**Problem.** `architecture-discovery/SKILL.md:83` ends its report with
"Documentation gaps and a recommended generation order." `architecture-bootstrap`
owns "sequencing architecture documentation work" (`:28`) and "Present the
intended documentation set and order" (`:55`). Bootstrap's step 1 ingests
Discovery's report — so it receives an ordering and produces a second, possibly
different one, with no reconciliation rule.

**Category.** Ownership/boundary issue
**Severity.** High

**Dependencies.** None strictly, but it edits `bootstrap` — **batch with F3 and
F20 into a single bootstrap pass** to avoid touching the file three times.

**Change direction.** Split along **evidence vs. plan**:
- Discovery reports gaps only — an unordered, evidence-grounded inventory
  ("these artifacts are missing relative to what the discovered architecture
  warrants"). Remove "and a recommended generation order" from `:83` and from
  step 4. Add "ordering or sequencing the work" to Discovery's
  `Not responsible for`.
- Bootstrap owns turning that inventory into a sequenced, user-confirmed plan.
  No change needed — it already claims this.

**Risk.** This makes Bootstrap thinner. `review.md` notes correctly that Bootstrap
is the plugin's only god-skill candidate and that the real risk is the opposite —
it becoming a thin wrapper. After F4 its unique value is *the plan and the user
checkpoint*, which is legitimate but must be visible. If F3 also removes
`ARCHITECTURE.md` content from it, verify Bootstrap still reads as a module with
its own responsibility rather than a preamble to Discovery. If it does not, that
is a signal to reconsider the Bootstrap/Discovery split in Phase 4 — **not** a
reason to leave the duplication in place now.

**Validation.**
- `grep -n "order" skills/architecture-discovery/SKILL.md` returns no output-producing use
- Discovery's `Not responsible for` names ordering
- Exactly one module claims sequencing

---

### F5 — arc42 §6/§7 point at C4 diagram types no module owns

**Problem.** `arc42-template.md:27` and `arc42-expert:55` annotate §6 Runtime View
with `→ sequence / C4 dynamic diagrams`. `c4-expert:52–60` declares exactly four
levels (Context, Container, Component, Code); neither Dynamic nor Deployment
appears in its scope, its workflow, or `c4-guidelines.md` — which mentions
neither word anywhere. §7 Deployment View carries no annotation at all.

**Category.** Ownership/boundary issue
**Severity.** High

**Dependencies.** None. Direction decided: **declare ownership, author content
later.**

**Change direction.** Adopt the levels/supplementary distinction — this is
correct C4 and the review's framing was wrong:
- Keep "the four levels" wording in `c4-expert` **unchanged**. Dynamic and
  Deployment are *not* levels; there is no "level 5."
- Add a **supplementary diagram types** entry to `c4-expert`'s
  `Responsible for` covering Dynamic and Deployment.
- Do **not** write Mermaid examples or guideline sections yet — deferred to
  Phase 4. This closes the ownership gap without expanding content.
- §6's dangling pointer resolves automatically once `c4-expert` claims Dynamic.
- §7 needs an explicit decision: either annotate it `→ C4 Deployment diagram`
  for symmetry, or leave it bare and say why. Note `arc42-expert:9` and `:79` and
  `README:74-75` all describe the C4 link set as "sections 3, 5, 6" — so §7's
  omission is currently *internally consistent*. Changing it means updating four
  locations, not one.

**Risk.** Declaring ownership without content creates a promise the module cannot
yet fulfill — a user routed to `c4-expert` for a deployment diagram gets scope
but no guidance. Mitigate by marking these explicitly as v0.1-declared /
v0.2-authored in the scope line itself, so the gap is stated rather than
discovered.

**Validation.**
- `c4-expert` scope names Dynamic and Deployment as supplementary, distinct from the four levels
- The phrase "the four levels" is unchanged
- No arc42 section annotation points at a diagram type no module claims
- §7's treatment is a stated decision, not an omission

---

### F6 — Two divergent C4 review checklists

**Problem.** `review-checklist.md:41-42` carries a one-line C4 entry (single zoom
level, labeled relationships) that defers to `c4-guidelines.md`.
`c4-guidelines.md:54-60` then carries its own five-bullet `## Review checklist`
containing two criteria the shared one omits: element count / readability, and
"Diagram matches the real system." `c4-expert:81-84` routes through the first to
reach the second.

**Category.** Consistency improvement (single source of truth)
**Severity.** Medium

**Dependencies.** **F6 must precede F7** — F7's defective line is
`c4-guidelines.md:60`, which F6 either moves or deletes. Doing F7 first means
editing a line that is about to relocate.

**Change direction.** Pick one home. Move the readability criterion up into
`review-checklist.md`'s C4 entry; handle the "matches real system" criterion per
F7. Delete `## Review checklist` from `c4-guidelines.md`, leaving that file
purely conventions + Mermaid examples. Simplify `c4-expert:81-84` to cite
`review-checklist.md` directly, matching how `adr-expert:74` and
`arc42-expert:81` already do it.

**Risk.** `review-checklist.md:3-6` justifies its own existence as preventing
"different, undocumented review bars." Consolidating in the wrong direction —
into `c4-guidelines.md` — would invert the plugin's governance layering and make
C4 the only artifact type with its own private review bar. Consolidate **up**,
into the shared checklist.

**Validation.**
- Exactly one `## Review checklist` heading exists under `references/`
- `c4-guidelines.md` contains conventions and examples only
- `c4-expert` reaches its review criteria in one hop
- No criterion is lost in the merge (5 bullets in, readability preserved, the fifth handled by F7)

---

### F7 — A reference file instructs `c4-expert` to violate its own scope

**Problem.** `c4-guidelines.md:60` reads "Diagram matches the real system
(cross-check with discovery)?" But `c4-expert:31` lists "discovering architecture
or system boundaries from source code" under `Not responsible for`, and its
workflow (`:70-78`) says architecture information is "never inferred from code
yourself, that's `architecture-discovery`'s job."

**Category.** Ownership/boundary issue
**Severity.** Medium

**Dependencies.** Depends on F6 (same line, same pass).

**Change direction.** Reword from a *check* to a *handoff*: if a diagram's
fidelity to the real system is in question, escalate to
`architecture-discovery`; do not verify against source code here. Fold into F6's
consolidation.

**Risk.** Deleting the criterion outright loses a real concern — diagram fidelity
does matter. The fix is relocating *who* answers it, not dropping it. Verify the
resulting escalation has a home in `c4-expert`'s `Escalates to`; it does
(`:35-37` already routes unknown boundaries to `architecture-discovery`), so this
should reuse that edge rather than invent one.

**Validation.**
- No reference file instructs a module to perform work its scope forbids
- The fidelity concern survives as an escalation, not a check
- `c4-expert` has an existing `Escalates to` edge covering it

---

### F8 — Librarian must detect drift it is forbidden the evidence for

**Problem.** `terminology.md:28-32` defines architecture drift as docs-vs-code
divergence, "Detected by `architecture-discovery` … not by
`architecture-librarian`, which only compares documentation artifacts against
each other." But `librarian:60-61` lists "**Obsolete decisions** — accepted ADRs
overtaken by reality but not marked deprecated/superseded," while `:70-72`
absolutely forbids reading application source code.

**Category.** Ownership/boundary issue
**Severity.** Medium

**Dependencies.** None.

**Change direction.** Narrow the bullet to what doc-to-doc evidence can support:
an accepted ADR **contradicted by a later accepted artifact** but not marked
deprecated/superseded. Drift-versus-code routes to `architecture-discovery`,
which `librarian:39-40` already correctly provides for.

**Risk.** Narrowing too far collapses the bullet into the existing "Architectural
conflicts" check (`:51-54`) and makes it redundant. The distinction to preserve:
*conflicts* = two artifacts disagree about a fact; *obsolete* = one artifact is
superseded in substance but not in status metadata. Keep the lifecycle-status
angle, drop the reality angle.

**Validation.**
- Every librarian check is answerable from documentation alone
- "Obsolete decisions" remains distinct from "Architectural conflicts"
- `terminology.md`'s drift definition needs no change to stay true

---

### F9 — The plugin's constitution is unreachable

**Problem.** `plugin-design-principles.md` has **zero** inbound citations from any
module, reference, or the README — the only first-order orphan among nine
references. It defines the Single-Responsibility rule and the exact three-part
boundary contract every module follows.

**Category.** Architecture improvement (governance)
**Severity.** Medium

**Dependencies.** README half depends on F1 (batch into the single README pass).
**F10, F14, and F21 all depend on F9** — writing rules into an unreachable file
achieves nothing.

**Change direction.** Give it two entry points: a "Contributing / adding a
module" section in the README, and a citation from `architecture-bootstrap` (the
module most likely to be extended, and per F20 the one currently citing no
references at all).

**Risk.** Low. The compounding cost of *not* fixing it is the real point: F3, F4,
F7, and F20 are precisely the violations this file exists to prevent, and it
could not prevent them because nothing points at it.

**Validation.**
- `plugin-design-principles.md` has ≥2 inbound citations
- It is reachable from the README in one hop
- The repo has no reference file with zero inbound citations

---

### F10 — No escalation termination rule

**Problem.** Nearly every escalation edge is bidirectional: discovery↔bootstrap,
discovery↔adr, discovery↔c4, discovery↔arc42, discovery↔librarian,
discovery↔reviewer, adr↔reviewer, adr↔librarian, reviewer↔librarian. No module
states a stopping condition. `reviewer:42-43` escalates to discovery when "the
proposal reflects an incomplete understanding of the existing system";
`discovery:42` escalates to reviewer when "architecture quality needs
evaluation" — a plausible live loop.

**Category.** Architecture improvement
**Severity.** Medium

**Dependencies.** **Depends on F9.** `plugin-design-principles.md` currently has
zero inbound citations; the only copy of a stopping rule cannot live in a file
nothing points at.

**Change direction.** Add a short "Escalation etiquette" section to
`plugin-design-principles.md`: escalate only when an ownership boundary is
actually crossed; escalate at most once per condition; if the module you
escalated to hands the same artifact back, stop and surface the ambiguity to the
user rather than re-escalating. Cite it from **at minimum the two agents**, which
are the pair with the live loop — not from all seven modules.

**Risk.** Over-application. Repeating the etiquette in every module bloats seven
files to fix a two-node problem, and duplicated governance text is itself a
drift source. Central rule + targeted citations. Conversely, if only the central
file gets it and F9's citations are dropped as redundant, the rule is invisible
again — both halves are required.

**Validation.**
- A stated termination condition exists in exactly one place
- Both agents cite it
- The reviewer↔discovery pair has a defined stopping party

---

### F11 — `severity-levels.md` claims an audience that does not use it

**Problem.** `severity-levels.md:3-4` scopes itself to reports by the librarian,
the reviewer, "or a skill's self-check." Its `## Usage` section (`:50-56`) lists
only the two agents. No skill cites the file — the three authoring skills cite
only `review-checklist.md` for self-review.

**Category.** Consistency improvement
**Severity.** Medium (in the plugin's own taxonomy: a *Possible inconsistency*)

**Dependencies.** None.

**Change direction.** Narrow, do not extend — inverting `review.md`'s
recommendation. The rule that makes the plugin coherent is: **authoring skills
get a verdict scale; auditors get a finding taxonomy.** Skills self-reviewing
should not classify the severity of their own defects. Delete `or a skill's
self-check` from `:4`. Nothing else changes.

**Risk.** Essentially none — this is a one-line edit that aligns the file with
what all seven modules already do. The rejected alternative (citing
`severity-levels.md` from three skills) would add three inbound edges to make a
sentence true, expanding the surface to fit the documentation rather than the
reverse.

**Validation.**
- The opening scope and `## Usage` name the same two consumers
- No skill cites `severity-levels.md`
- `review-checklist.md`'s verdict scale remains the skills' self-review vocabulary

---

### F12 — Reviewer cannot hand off two of the artifact types it reviews

**Problem.** `reviewer:47` reviews "ADRs, C4 diagrams, arc42 sections, or
freeform design proposals," but `:37-43` escalates only to
`architecture-librarian`, `adr-expert`, and `architecture-discovery`. No edge to
`c4-expert` or `arc42-expert`.

**Category.** Ownership/boundary issue (escalation asymmetry)
**Severity.** Low

**Dependencies.** **Depends on F10.** Adding edges to a near-complete
bidirectional mesh before a stopping rule exists makes the loop risk worse.

**Change direction.** Add two bullets mirroring the existing `adr-expert` one,
**conditionally phrased**: `c4-expert` when the architecture is sound but the
diagram needs rework; `arc42-expert` when the narrative documentation is the weak
part. Conditional wording is the existing house style — every current edge names
its trigger.

**Risk.** Mesh growth. At 7 modules the bidirectional graph is manageable; the
concern is the trajectory. Mitigated by F10 landing first and by keeping each
edge conditional rather than a bare arrow.

**Validation.**
- Every artifact type the reviewer accepts has a corresponding authoring handoff
- Each new bullet names its trigger condition
- No new edge is unconditional

---

### F13 — `adr-expert` cannot route structural consequences

**Problem.** `adr-expert:31-37` escalates to discovery, reviewer, and librarian —
but not to `c4-expert` or `arc42-expert`. The reverse edges exist
(`arc42-expert:36` → adr; `c4-expert:42` → arc42). Meanwhile
`librarian:56-58` explicitly reports "an accepted ADR with no diagram reflecting
it" as a coverage gap and recommends `c4-expert` — so the plugin already knows
this handoff is needed but routes it through an audit pass.

**Category.** Ownership/boundary issue (escalation asymmetry)
**Severity.** Low

**Dependencies.** Depends on F10, same reasoning as F12.

**Change direction.** Add `c4-expert` and `arc42-expert` as **conditional** edges.
The condition matters more here than in F12: not every ADR implies a diagram —
"choose a logging library" does not. Phrase as: `c4-expert` when the decision
changes structure a documented view should show; `arc42-expert` when §9 needs to
cite the new ADR. Automatic ADR→C4 routing would be wrong.

**Risk.** Unconditional edges here would push `adr-expert` toward proposing
diagrams for routine decisions, contradicting `severity-levels.md:20-22` ("Not
every container or component needs an ADR — only flag a missing ADR where the
diagram represents a real decision"). The conditionality is load-bearing.

**Validation.**
- Both edges are conditional and name the trigger
- A logging-library ADR does not trigger either
- The librarian's coverage-gap check remains a backstop, not the only path

---

### F14 — Module naming convention is unwritten

**Problem.** Two conventions coexist: `architecture-bootstrap` /
`architecture-discovery` (prefixed) versus `adr-expert` / `c4-expert` /
`arc42-expert` (suffixed). Both agents use the `architecture-` prefix. There is a
defensible latent logic — `architecture-*` = cross-cutting workflow, `*-expert` =
owns one notation — but it is written down nowhere.

**Category.** Future scalability
**Severity.** Low

**Dependencies.** Depends on F9. Lands in the same `plugin-design-principles.md`
pass as F10 to avoid editing that file twice.

**Change direction.** State the rule in `plugin-design-principles.md`:
`<notation>-expert` for modules owning a single documentation format;
`architecture-<verb>` for cross-cutting modules. Keep all current names — they
already conform.

**Risk.** None to current names. The risk is deferral: the roadmap names DDD,
RFC, and threat-modeling modules, each of which could plausibly take either form.
Settling the rule now is far cheaper than renaming shipped modules later.

**Validation.**
- The rule is stated and all 7 existing names conform without renaming
- The rule assigns an unambiguous form to each roadmap module

---

### F15 — `adr-expert` is the only module with no status marker

**Problem.** Six of seven modules carry `> Status: v0.1 stub`. `adr-expert` has
none (would sit after `:13`). `README:9-11` says "The skill/agent bodies marked
*stub* define their workflow and conventions" — so the omission reads as a claim
that `adr-expert` is production-ready, while `README:91-92` lists it among
modules to be deepened in v0.2.

**Category.** Documentation drift
**Severity.** Low

**Dependencies.** None. Zero-risk one-line addition.

**Change direction.** Add the marker. Note the existing markers use two forms —
skills say `v0.1 stub — structure and workflow are defined; deepen during use`,
agents say `v0.1 stub.` — so match the skill form.

**Risk.** None, unless `adr-expert` genuinely is more mature, in which case mark
it explicitly (`v0.1 — deepened`) so the distinction is intentional rather than
absent. The README roadmap suggests it is not.

**Validation.** `grep -L '^> Status:' skills/*/SKILL.md agents/*.md` returns nothing.

---

### F16 — Six of nine references are invisible from the README

**Problem.** `README:37-40` and the layout block `:57-60` list only
`adr-template.md`, `arc42-template.md`, and `c4-guidelines.md` — omitting
`terminology.md`, `review-checklist.md`, `severity-levels.md`,
`quality-attributes.md`, `decision-drivers.md`, and
`plugin-design-principles.md`. The omitted set includes the two most-cited files
in the plugin. The heading "References (shared templates)" is also too narrow —
most are vocabularies and taxonomies, not templates.

**Category.** Documentation drift
**Severity.** Low

**Dependencies.** Depends on F1 — same file, same pass.

**Change direction.** List all nine grouped by kind (templates / vocabularies /
governance) and widen the heading accordingly. Update the layout block to match.

**Risk.** Low. Grouping by kind is worth doing rather than listing nine flat —
the grouping is what makes the reference layer legible and it sets up F21 and
the Phase 5 reference-ownership rule.

**Validation.**
- All nine references appear in the README, grouped
- The layout block matches `ls references/`
- The heading covers non-template content

---

### F17 — One bare-filename citation

**Problem.** `arc42-template.md:38` cites `adr-template.md` without the
`references/` prefix. Verified: this is the **only** such deviation among all
citations in the repo; every other cross-reference uses the full path.

**Category.** Consistency improvement
**Severity.** Low

**Dependencies.** None.

**Change direction.** Change to `references/adr-template.md`.

**Risk.** None. Currently resolvable, so this is pure hygiene — but bare
filenames are what break first if references are ever reorganized into
subdirectories.

**Validation.** `grep -rnE '`[a-z0-9-]+\.md`' references/ skills/ agents/` returns
no bare reference filenames.

---

### F18 — The plugin's own boundary is unstated

**Problem.** No module owns *designing* a target architecture.
`discovery:33` explicitly disclaims it; the reviewer critiques but does not
design; the three authoring skills document decisions already made. `adr-expert`
comes closest and will attract the request by proximity — then apply an
ADR-shaped workflow to a question that is not yet a decision.

**Category.** Documentation drift (unstated boundary)
**Severity.** Low

**Dependencies.** Depends on F1 — same file, same pass.

**Change direction.** State it once in the README: this toolkit documents,
reviews, and governs architecture; it does not design it. **No module change.**

**Risk.** The tempting over-reaction is adding a design module. Do not — see
"Changes that should NOT be done." A documentation toolkit that silently starts
inventing architecture violates the plugin's core principle that skills propose
and document rather than invent.

**Validation.**
- The README states the plugin-level boundary explicitly
- No module gained a design responsibility

---

### F19 — Unexplained `WebFetch` grant on the reviewer

**Problem.** `reviewer:10` grants `Read, Grep, Glob, WebFetch`. Nothing in the
body uses or justifies network access; the description calls the agent
"Read-only" and `:78` says "Do not edit the artifact." The librarian, doing
comparable work, gets `Read, Grep, Glob`.

**Category.** Correctness issue (capability hygiene)
**Severity.** Low

**Dependencies.** None, but requires a user decision — I cannot infer intent.

**Change direction.** Plausibly deliberate: the Evidence dimension the reviewer
is told to push hardest on (`:51-56`) could reasonably involve verifying a cited
RFC, spec, or vendor claim. If so, say so in `## How you review`. If not, drop
the grant.

**Risk.** Dropping it silently could remove a capability the design intends.
Keeping it undocumented leaves an unexplained network privilege in an otherwise
tightly-scoped read-only agent. Either resolution is fine; leaving it ambiguous
is not. **Ask before changing.**

**Validation.** The grant is either justified in the body or removed — no
unexplained capability remains.

---

### F20 — Bootstrap cites zero shared references *(new)*

**Problem.** `architecture-bootstrap` is the only module citing **no**
`references/*.md` file. It never points at `terminology.md`, despite coordinating
every other module and being the module that introduces `ARCHITECTURE.md`.

**Category.** Consistency improvement
**Severity.** Medium

**Dependencies.** Batch with F3 and F4 (same file). The `plugin-design-principles`
citation half also satisfies F9.

**Change direction.** Anchor Bootstrap to the shared layer: cite
`terminology.md` where it names artifact types (which is where F3's
`ARCHITECTURE.md` definition will live), and `plugin-design-principles.md` in the
context of extending the plugin.

**Risk.** Low. This is the structural fix behind F3 rather than a cosmetic one:
the module least anchored to the glossary is the one that invented an artifact
the glossary does not define. Adding citations without F3's terminology entry
would point at a glossary that still lacks the relevant term — do F3 first.

**Validation.**
- Bootstrap cites ≥1 reference file
- No module cites zero references
- Its `ARCHITECTURE.md` mention resolves to the glossary definition

---

### F21 — `decision-drivers.md` is invisible to every module *(new)*

**Problem.** `decision-drivers.md` is cited by `adr-template.md:30` and
`review-checklist.md:15` — but by **zero** skills or agents, and it is absent
from the README. It is reachable transitively and unreachable in practice. Note
`adr-expert` cites neither it nor `quality-attributes.md`, despite ADR
authoring being exactly where decision drivers apply.

**Category.** Consistency improvement
**Severity.** Low

**Dependencies.** Depends on F16 (README listing) and F9 (reference-reachability
principle).

**Change direction.** Two options, needs a decision: (a) accept transitive
reachability as sufficient — `adr-expert` reaches it via `adr-template.md` — and
document that as the intended pattern; or (b) cite it directly from
`adr-expert`'s "Gather context" step. Option (a) is more consistent with the
plugin's reference-don't-duplicate posture; option (b) is more discoverable.
**Lean (a)**, plus the README listing from F16.

**Risk.** Option (b) starts a pattern where every module cites every relevant
reference, inflating the citation graph. The plugin's existing design deliberately
uses transitive reachability (templates cite vocabularies; skills cite
templates). Preserve that.

**Validation.**
- Every reference file is reachable from the README
- The intended reachability pattern (direct vs. transitive) is stated once
- No reference has zero inbound citations of any kind

---

## Phase 0 — Safety baseline

**Goal.** A checkpoint we can return to, and a repeatable way to tell whether a
later phase broke something.

This matters more than usual here: **there are no tests and no CI.** Every
"validation" in this plugin is a human or an agent reading Markdown. Without a
baseline, a regression is invisible.

**Checkpoint strategy**
- Confirm the working tree is clean (it is, except untracked `review.md`).
- Decide `review.md`'s fate: commit it as the review of record, or leave it
  untracked. It is 481 lines of analysis and currently invisible to git.
- Tag the pre-change state (e.g. `v0.1-pre-cleanup`) — the plugin is on `master`
  with `origin/master` present.
- Work on a branch, not `master`, one commit per phase.

**Validation commands** (prototyped and run — current results recorded above)
1. **Boundary-block lint** — every `SKILL.md` and agent has all three headings. *Currently: 7/7 pass.*
2. **Reference-path lint** — every `references/*.md` path mentioned resolves. *Currently: 9/9 pass.*
3. **Module-name lint** — every backticked `architecture-*` / `*-expert` identifier resolves to a real skill dir or agent file. *Currently: 1 failure (`architecture-documentation`).*
4. **Status-marker lint** — every module has a `> Status:` line. *Currently: 1 failure (`adr-expert`).*
5. **Fan-in report** — inbound citation count per reference file. *Currently: 1 zero (`plugin-design-principles.md`), 1 module-zero (`decision-drivers.md`).*

Checks 1, 2, and 5 must **stay** passing through every phase. Checks 3 and 4 must
**flip** to passing in Phase 1.

**Documentation snapshot.** Record the current fan-in table and boundary-block
inventory (both captured in this plan) so post-change drift is measurable rather
than remembered.

---

## Phase 1 — Critical correctness fixes

**Items:** F1, F16, F18, F9 *(README half only)*, F2, F15, F17

**Why these come first.** Everything here is a statement the plugin currently
makes that is false. F1 is the only defect a user hits *before* reading anything
else — it is in the module table, the directory layout, and the greenfield entry
point, and it names a module that has never existed in any commit. F2 is the only
defect that produces a runtime failure rather than a doc error: an approved index
regeneration hits a tool-unavailable error.

Nothing in Phase 1 changes a boundary. That is deliberate — it means Phase 1 can
land and be verified without any architectural judgment, establishing a correct
baseline before Phase 2 starts moving responsibilities.

**One-pass rule for the README.** F1, F16, F18, and F9's README half all edit
`README.md`. Do them as **one** pass. The README's problem is not four separate
bugs — it is a file written before the plugin reached its current form and never
revisited. Piecemeal patching reproduces exactly that.

F15 and F17 are one-line, zero-risk riders included here because they flip a
baseline lint (F15) and remove the single deviation from an otherwise perfect
convention (F17).

**Exit criteria.** Lints 3 and 4 pass. No file asserts a capability its
frontmatter does not grant. The README's workflow section is re-derived from
actual `Escalates to` blocks, not patched. **The user has updated the stale
additional-working-directory setting** — outside the repo, so it cannot be
verified by lint.

---

## Phase 2 — Ownership and boundary cleanup

**Items:** F3, F4, F20 *(one bootstrap pass)*; F5; F8; F9 *(bootstrap half)*; F10; F12; F13

**Goal.** Every module answers cleanly: what do I own, what do I explicitly not
own, where do I send work.

**Internal ordering is load-bearing:**
1. **F3 terminology entry first**, then `bootstrap:56`. Glossary before module,
   or the two disagree mid-phase.
2. **F3 + F4 + F20 + F9's bootstrap citation = one bootstrap commit.** Four items,
   one file, one pass.
3. **F10 before F12/F13.** A stopping rule must exist before more edges are added
   to a near-complete bidirectional mesh.
4. **F9's bootstrap half before F10**, since F10's rule lives in the file F9 makes
   reachable.

**Watch item.** After F3 and F4, re-read `architecture-bootstrap` end to end and
ask honestly whether it still reads as a module with its own responsibility. It
loses `ARCHITECTURE.md` content and its generation order in the same phase. If it
reads as a thin wrapper, that is a Phase 4 input — **not** a reason to reverse
these fixes.

**Exit criteria.** `ARCHITECTURE.md` has exactly one named owner. Exactly one
module claims sequencing. No arc42 annotation points at an unowned diagram type.
Every librarian check is answerable from documentation alone. A termination rule
exists and both agents cite it. Every artifact type the reviewer accepts has a
conditional authoring handoff.

---

## Phase 3 — Reference architecture cleanup

**Items:** F6, F7 *(paired)*; F11; F14; F21; F19 *(pending user decision)*

**Goal.** One concept, one home.

Phase 3 is where the root cause gets addressed rather than its symptoms. The
module layer has a template every file follows; the reference layer has nothing.
That asymmetry is why the C4 checklist forked (F6), why a boundary violation
survived inside a guidelines file (F7), and why a file claims an audience that
does not exist (F11).

**Ordering:** F6 strictly before F7 (same line). F14 batches with F10's
`plugin-design-principles.md` edit if Phase 2 and 3 land close together —
otherwise that file gets edited twice.

**The durable output** is not the four edits; it is the **reference-ownership
rule** written into `plugin-design-principles.md`: who may add a reference file,
what belongs in a template vs. a vocabulary vs. a governance doc, and the rule
that a criterion lives in exactly one file. Without that, Phase 3 gets re-done at
v0.3.

**Exit criteria.** Exactly one `## Review checklist` under `references/`. No
reference instructs a module to violate its scope. Every reference's stated
audience matches its actual citations. No reference has zero inbound citations.
The naming convention is written and all current names conform.

---

## Phase 4 — Workflow optimization *(deferred)*

**Goal.** Make `Bootstrap → Discovery → Authoring → Review → Maintenance` legible
without introducing hidden orchestration.

Deferred deliberately: Phases 1–3 change what modules *claim*. Phase 4 would
change what they *do* — and it should be informed by the post-Phase-2 shape of
Bootstrap, which is exactly what the Phase 2 watch item is measuring.

Candidate scope: the Bootstrap/Discovery entry-point ambiguity (both are
defensible starting points and each escalates to the other); authoring-flow
consistency across the three `*-expert` skills; F5's deferred content (Dynamic
and Deployment guidelines with Mermaid examples matching the existing
Context/Container ones); deepening the six v0.1 stubs.

**Constraint that must survive.** `bootstrap:75-82` and `README:66-68` both state
honestly that skills cannot invoke each other and this is sequencing, not
orchestration. Any workflow improvement that blurs this reintroduces the
anti-pattern the plugin currently avoids cleanly.

---

## Phase 5 — Future scalability *(deferred)*

**Goal.** Make DDD, RFC, and threat-modeling modules cheap and safe to add.

Deliverables: the naming convention (landed early in Phase 3, since it is nearly
free there); explicit extension rules in `plugin-design-principles.md`; a
documented reference-ownership rule; and **validation automation**.

**On the lint** — deferred per decision, and I think that is right. The four
checks are prototyped and run in ~15 lines of shell; checks 3 and 4 would have
caught F1 and F15 outright. But a 5-skill plugin with no CI does not need
automation infrastructure before the roadmap modules land, and building it now
means maintaining a lint against a spec that Phases 2–3 are still changing.
Revisit when the first new module is added — that is the moment the checks start
paying for themselves.

**Scaling risk to watch.** Not module count — the escalation graph. At 7 modules
the near-complete bidirectional mesh is manageable. At 10–12 it needs either the
F10 etiquette or explicit tiering (extraction → authoring → governance)
constraining which layers may escalate to which.

---

## Recommended execution order

1. **Phase 0** — branch, tag, record baseline lint results.
2. **F1 + F16 + F18 + F9(README)** — one README commit.
3. **F2** — librarian read-only reconciliation.
4. **F15 + F17** — zero-risk one-liners; flips a baseline lint.
5. **F3 terminology entry** — define `ARCHITECTURE.md` ownership.
6. **F3 + F4 + F20 + F9(bootstrap)** — one bootstrap/discovery commit.
7. **F5** — declare C4 supplementary diagram ownership; decide §7.
8. **F8** — narrow the librarian's obsolete-decisions check.
9. **F10 (+ F14)** — escalation etiquette and naming rule in `plugin-design-principles.md`.
10. **F12 + F13** — conditional escalation edges.
11. **F6 → F7** — consolidate the C4 checklist, convert the fidelity check to a handoff.
12. **F11** — narrow `severity-levels.md`'s stated audience.
13. **F21 + reference-ownership rule** — close the reference layer.
14. **F19** — resolve `WebFetch` (needs a decision).
15. **Re-run all five lints; re-derive the fan-in table; compare to the Phase 0 snapshot.**

**Why this order minimizes risk.**

- **Truth before boundaries.** Steps 2–4 fix statements that are simply false, in
  files where no judgment is required. If something goes wrong later, we roll
  back to a state that is at least accurate.
- **Glossary before modules.** Step 5 before step 6: define the term, then make
  the module conform. The reverse produces a module that conforms to nothing.
- **One file, one pass.** Steps 2, 6, and 9 each batch multiple items into a
  single file edit. Touching `README.md` four times, `bootstrap` four times, or
  `plugin-design-principles.md` twice is how the drift being repaired originally
  happened.
- **Reachability before rules.** Step 9's etiquette depends on step 6 having made
  its host file reachable. Writing rules into an orphan file is a no-op.
- **Stopping rule before more edges.** Step 9 before step 10. Adding four
  escalation edges to an already-bidirectional graph without a termination
  condition makes the one real loop risk worse.
- **Consolidate before rewording.** Step 11's F6 relocates the exact line F7
  rewords. Reversed, we edit a line that is about to move.
- **Decisions last.** Step 14 needs user intent and should not block mechanical work.

---

## Changes that should NOT be done

**Grant the librarian `Edit`/`Write` (the tempting F2 fix).** It would make the
librarian the only module able to silently mutate the documentation set it
audits, and reduce the guardrail from a capability boundary to a prompt-level
approval gate. Reword the contract instead. *Rejected.*

**Add `severity-levels.md` citations to the three authoring skills (`review.md`'s
F11 recommendation).** This expands the citation graph to make a sentence true.
The plugin's actual rule — authoring skills get a verdict scale, auditors get a
finding taxonomy — is better, and it is already what all seven modules do.
*Inverted: narrow the file instead.*

**Add "level 5 / level 6" for Dynamic and Deployment.** C4 has four levels.
Dynamic and Deployment are supplementary diagram types. Keep `c4-expert`'s "the
four levels" wording exactly as it is. *Rejected as framed; adopt the
supplementary-types framing instead.*

**Make ADR→C4 and ADR→arc42 escalations unconditional (F13).** Not every ADR
implies a diagram — "choose a logging library" does not. Unconditional edges
would push `adr-expert` toward proposing diagrams for routine decisions,
contradicting `severity-levels.md:20-22`. *Accept conditionally only.*

**Repeat the escalation etiquette in all seven modules (F10 as written).** Seven
copies of a governance rule to fix a two-node loop, and duplicated governance
text is itself a drift source. Central rule, cite from the two agents.
*Scope reduced.*

**Assign `ARCHITECTURE.md` to `arc42-expert` (`review.md`'s preferred F3 fix).**
`ARCHITECTURE.md` is an entry point and navigation map; arc42 is deep
architecture documentation. Making arc42-expert own it either bloats arc42's
scope or turns `ARCHITECTURE.md` into a second arc42. *Rejected in favor of the
navigation-map framing.*

**Build the CI lint now.** Cheap and it would have caught F1 — but the spec it
would enforce is still being changed by Phases 2–3. *Postponed to Phase 5,
revisit when the first roadmap module lands.*

**Add a `plugin-architecture-reviewer` module.** Premature at 7 modules.
`plugin-design-principles.md` plus the lint covers it. *Postponed indefinitely.*

**Add a design module to close F18.** A documentation toolkit that starts
inventing architecture violates its own core principle. State the boundary in the
README; add nothing. *Rejected.*

**Force the escalation graph into a strict DAG.** Architectural work is genuinely
cyclic — Discovery → ADR → Review → Discovery is a legitimate loop. The fix is
termination rules, not acyclicity. *Rejected.*

**Restructure, rename, or split any module.** Nothing in 21 findings requires it.
*Out of scope.*

---

## Final recommendation

**Preserve the current architecture.** The seven-module split, the three-part
boundary contract, the reference-don't-duplicate rule, and the honest
"sequencing, not orchestration" disclaimer are all correct and unusually
disciplined for a v0.1. No finding argues for restructuring. The uniformity of
the boundary block across all 7 modules is what made this review tractable at all
— it should be protected, not revised.

**No restructuring is needed.** Every item is a targeted repair to a statement,
an ownership label, or a citation. Zero module boundaries move. Zero modules are
added, removed, split, or renamed. The largest single change is deleting a
six-line duplicated checklist.

**Priority.**
- **Critical, do now:** F1 — the README's front door names a module that has never existed, and the stale name is live in environment config, so it is propagating rather than decaying.
- **High, Phase 1–2:** F2 (only runtime-breaking defect), F3, F4, F5, F20.
- **Medium, Phase 2–3:** F6, F7, F8, F9, F10, F11.
- **Low, opportunistic:** F12–F19, F21.

**The framing worth keeping.** The one-line diagnosis is better than a
finding-by-finding list:

> The module architecture is right. The plugin has no governance for its own
> architecture.

That explains the *pattern* in F3/F4/F6/F7/F20 rather than treating them as five
separate defects — and it is why Phase 3's durable output is the
reference-ownership rule, not the four edits it produces.

---

## Verification

No code, no tests, no build. Verification is the five lints plus targeted reading.

**After every phase**
1. Run all five Phase 0 lints. Checks 1, 2, 5 stay green; 3 and 4 flip green in Phase 1.
2. `git diff --stat` against the phase's starting commit — confirm only intended files changed.

**After Phase 1**
- `grep -r "architecture-documentation" --include='*.md' .` → hits only in `review.md`
- `grep -n "architecture-bootstrap" README.md` → non-empty (currently zero hits)
- Read `README.md` top to bottom as a new user; every named module resolves, every reference listed exists
- Confirm the user has fixed the additional-working-directory setting

**After Phase 2**
- `grep -rn "ARCHITECTURE.md" skills/ agents/ references/` — exactly one owner, no content claims
- `grep -rn "order" skills/architecture-discovery/SKILL.md` — no output-producing use
- Trace reviewer→discovery→reviewer and confirm a documented stopping party
- Re-read `architecture-bootstrap` end to end (watch item)

**After Phase 3**
- `grep -rn "## Review checklist" references/` — exactly one hit
- Re-derive the fan-in table; compare against the Phase 0 snapshot; no reference at zero
- Trace `c4-expert` → review criteria in one hop

**End-to-end smoke test.** Invoke `architecture-bootstrap` against a small
throwaway repo and follow its plan by hand through discovery → one ADR → one C4
diagram → librarian. Confirm no step routes to a module whose scope excludes the
work, and no artifact is produced without a named owner. This is the only test
that exercises the escalation graph as a user would, and it is what would have
surfaced F1, F4, and F5 without a line-by-line audit.
