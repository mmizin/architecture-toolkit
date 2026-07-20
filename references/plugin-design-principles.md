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
