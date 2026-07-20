# Architecture Toolkit

A Claude Code **plugin** for software architecture documentation and decision
management. It bundles focused skills and independent-reviewer agents behind one
namespace, so a team can discover architecture from existing code, generate and
govern ADRs, draw C4 diagrams, and maintain arc42 documentation — consistently
and over time.

> **Status: v0.1 — skeleton.** The plugin structure, all modules, and shared
> templates are in place. The skill/agent bodies marked *stub* define their
> workflow and conventions; they get deepened through use.

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
| `architecture-documentation` | Orchestrate an end-to-end documentation set for a project ("document everything"). |
| `adr-expert` | Create, review, improve, and govern ADRs across their lifecycle. |
| `c4-expert` | Create and review C4 diagrams (Context / Container / Component). |
| `arc42-expert` | Generate, update, and validate arc42 architecture documentation. |

### Agents (independent roles)
| Agent | Purpose |
|-------|---------|
| `architecture-reviewer` | Independent senior-architect critique of an ADR/diagram/doc. |
| `architecture-librarian` | Audits the docs repository for duplicates, conflicts, obsolete decisions, and index health. |

### References (shared templates)
- `references/adr-template.md`
- `references/arc42-template.md`
- `references/c4-guidelines.md`

## Layout

```
architecture-toolkit/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   ├── architecture-discovery/SKILL.md
│   ├── architecture-documentation/SKILL.md
│   ├── adr-expert/SKILL.md
│   ├── c4-expert/SKILL.md
│   └── arc42-expert/SKILL.md
├── agents/
│   ├── architecture-reviewer.md
│   └── architecture-librarian.md
├── references/
│   ├── adr-template.md
│   ├── arc42-template.md
│   └── c4-guidelines.md
└── README.md
```

## Recommended workflow

Claude Code skills cannot call each other programmatically, so this toolkit is a
set of **independent** modules with a recommended order rather than a runtime
orchestrator:

1. **`architecture-discovery`** — scan the project; get a report of structure,
   data flows, doc gaps, and candidate decisions.
2. **`adr-expert`** — turn the significant candidate decisions into ADRs.
3. **`c4-expert`** — draw the Context and Container diagrams.
4. **`arc42-expert`** — assemble the sections into full documentation, linking
   ADRs (§9) and C4 diagrams (§3/5/6).
5. **`architecture-reviewer`** (agent) — get an independent critique.
6. **`architecture-librarian`** (agent) — keep the repository consistent over
   time.

For a greenfield or undocumented project, start at
**`architecture-documentation`**, which sequences the above into one plan.

## Usage

Once installed as a Claude Code plugin, invoke a skill by name (e.g. ask Claude
to "use adr-expert to draft an ADR for …") or trigger it implicitly by
describing the task. Agents are launched as subagents for review/audit tasks.

## Roadmap

- **v0.1** — plugin skeleton, all modules + templates (this release).
- **v0.2** — deepen `architecture-discovery` and `adr-expert`; realistic
  end-to-end examples.
- **v0.3+** — MCP integrations (GitHub / Jira / Confluence) so the librarian can
  verify ADR links against real trackers and wikis; additional modules (DDD,
  RFC, threat modeling).

## License

MIT
