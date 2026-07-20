# Plugin Design Principles

Read this before adding a new skill or agent, or a new capability to an
existing one.

## Single Responsibility, at the plugin level

Each skill and agent owns one well-defined responsibility. Before adding a
capability, ask:

**Does this belong here?**

- If it duplicates what another module already owns, extend that module or
  delegate to it — don't duplicate the capability in two places.
- If it doesn't fit any existing module's responsibility, it's a candidate
  for a new module, not a bolt-on to the nearest one.
- When in doubt, prefer delegation over duplication.

## How a module should state its boundary

Every skill/agent description should make it possible to answer, without
reading the body: what does this do, and what does it explicitly not do?
Where a module's boundary is easy to confuse with another's, state it as:

- **Responsible for** — the capability this module owns.
- **Not responsible for** — capabilities that sound related but belong
  elsewhere.
- **Escalates to** — which other module to hand off to, and when.

## Escalation etiquette

Escalation edges in this plugin are deliberately bidirectional — architectural
work is genuinely cyclic, and forcing the graph into a DAG would misrepresent
it. What keeps that safe is a stopping rule, not acyclicity:

- **Escalate only when an ownership boundary is actually crossed.** Needing more
  context is not an escalation; needing work that your `Not responsible for`
  block names is.
- **Escalate at most once per condition.** One handoff per distinct question.
- **Never re-escalate a returned artifact.** If the module you escalated to
  hands the same artifact back unchanged, stop. Surface the ambiguity to the
  user — state what each module believes it does not own — and let them decide.
  A second hop on the same condition is a loop, not diligence.
- **Name the trigger on every edge.** Bare arrows invite reflexive handoffs;
  conditional edges document when *not* to escalate too.

## Naming convention

Two forms, chosen by what the module owns:

- **`<notation>-expert`** — the module owns a single documentation format and
  its conventions: `adr-expert`, `c4-expert`, `arc42-expert`.
- **`architecture-<verb>`** — the module performs a cross-cutting activity over
  whatever formats are present: `architecture-discovery`,
  `architecture-bootstrap`, `architecture-reviewer`, `architecture-librarian`.

All current names conform; none need renaming. Applied to the roadmap: DDD and
threat modeling are activities spanning several artifact types
(`architecture-*`), while RFCs are a single document format (`rfc-expert`). If a
proposed name fits neither form, that usually means the module owns more than
one responsibility — revisit the split before revisiting the name.
