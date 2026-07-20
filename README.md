# Architecture Toolkit

A Claude Code **plugin** for software architecture documentation and decision
management. It bundles focused skills and independent-reviewer agents behind one
namespace, so a team can discover architecture from existing code, generate and
govern ADRs, draw C4 diagrams, and maintain arc42 documentation — consistently
and over time.

> **Status: v0.1 — initial implementation.** The plugin structure, all
> modules, and shared templates are in place. Workflows and governance are
> defined; deeper examples and operational guidance come through use.

## Why a plugin (not one big skill)

A single monolithic "architecture skill" becomes an unmaintainable monster as
you add C4, arc42, DDD, RFC, threat modeling, and so on. A Claude Code plugin is
purpose-built as a **container** for reusable components — skills, agents,
references (and later, MCP servers). Each module owns exactly one area.

## What's inside

### Skills (workflows)
| Skill | Purpose |
|-------|---------|
| `architecture-discovery` | Scan a codebase, reconstruct its architecture, and surface missing docs and candidate ADRs. |
| `architecture-bootstrap` | Sequence an end-to-end documentation set for a project ("document everything"). |
| `adr-expert` | Create, review, improve, and govern ADRs across their lifecycle. |
| `c4-expert` | Create and review C4 diagrams (Context / Container / Component, plus supplementary Dynamic and Deployment views). |
| `arc42-expert` | Generate, update, and validate arc42 architecture documentation. |

### Agents (independent roles)
| Agent | Purpose |
|-------|---------|
| `architecture-reviewer` | Independent senior-architect critique of an ADR/diagram/doc. |
| `architecture-librarian` | Audits the docs repository for duplicates, conflicts, obsolete decisions, and index health. |

### References (shared templates, vocabularies, governance)

**Templates** — the shape an artifact takes:
- `references/adr-template.md`
- `references/arc42-template.md`
- `references/c4-guidelines.md`

**Vocabularies** — the shared terms modules reason with:
- `references/terminology.md`
- `references/quality-attributes.md`
- `references/decision-drivers.md`

**Governance** — how work is judged and how the plugin itself is built:
- `references/review-checklist.md`
- `references/severity-levels.md`
- `references/plugin-design-principles.md`

## Layout

```
architecture-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── architecture-bootstrap/SKILL.md
│   ├── architecture-discovery/SKILL.md
│   ├── adr-expert/SKILL.md
│   ├── c4-expert/SKILL.md
│   └── arc42-expert/SKILL.md
├── agents/
│   ├── architecture-reviewer.md
│   └── architecture-librarian.md
├── references/
│   ├── adr-template.md
│   ├── arc42-template.md
│   ├── c4-guidelines.md
│   ├── decision-drivers.md
│   ├── plugin-design-principles.md
│   ├── quality-attributes.md
│   ├── review-checklist.md
│   ├── severity-levels.md
│   └── terminology.md
└── README.md
```

## Recommended workflow

Claude Code skills cannot call each other programmatically, so this toolkit is a
set of **independent** modules with a recommended order rather than a runtime
orchestrator:

1. **`architecture-discovery`** — scan the project; get a report of structure,
   data flows, doc gaps, and candidate decisions. This is the grounding step:
   the discovery report is the preferred input for the authoring modules
   below, though each can also work from an ADR, diagram, or description the
   user provides directly.
2. **Authoring** — discovery hands off to whichever artifacts the gaps call for:
   - **`adr-expert`** — turn the significant candidate decisions into ADRs.
   - **`c4-expert`** — diagram the discovered boundaries (Context / Container).
   - **`arc42-expert`** — write the architecture narrative, linking ADRs (§9)
     and C4 diagrams (§3/5/6/7).
3. **`architecture-reviewer`** (agent) — independent critique. Every authoring
   module escalates here when the question is architectural quality rather than
   documentation form.
4. **`architecture-librarian`** (agent) — keep the repository consistent over
   time. Every authoring module escalates here for cross-artifact consistency.

For a greenfield or undocumented project, start at
**`architecture-bootstrap`**, which grounds itself in `architecture-discovery`
and sequences the above into one plan you confirm before any artifact is written.

| Starting point | Flow |
|---|---|
| Existing, partially documented codebase | `architecture-discovery` → authoring modules → `architecture-reviewer` → `architecture-librarian` |
| Undocumented / greenfield project | `architecture-bootstrap` → `architecture-discovery` → plan → authoring modules → `architecture-librarian` |

## What this toolkit does not do

This toolkit **documents, reviews, and governs** architecture. It does not
**design** it. No module invents a target architecture, chooses between
candidate designs, or decides on your behalf — the skills capture and critique
decisions that people make. If you need a design produced rather than recorded,
that work happens outside this plugin; bring the outcome back here to document.

## Contributing / adding a module

Read `references/plugin-design-principles.md` first. It defines the
Single-Responsibility rule and the three-part boundary contract
(`Responsible for` / `Not responsible for` / `Escalates to`) that every skill and
agent in this plugin follows. A new module that does not carry that block, or
that overlaps an existing module's ownership, is not ready to add.

## Usage

Once installed as a Claude Code plugin, invoke a skill by name (e.g. ask Claude
to "use adr-expert to draft an ADR for …") or trigger it implicitly by
describing the task. Agents are launched as subagents for review/audit tasks.

## Roadmap

- **v0.1** — plugin skeleton, all modules + templates (this release).
- **v0.2** — deepen `architecture-discovery` and `adr-expert`; `c4-expert`
  conventions and examples for Dynamic/Deployment views; realistic
  end-to-end examples.
- **v0.3+** — MCP integrations (GitHub / Jira / Confluence) so the librarian can
  verify ADR links against real trackers and wikis; additional modules (DDD,
  RFC, threat modeling).

## License

MIT
