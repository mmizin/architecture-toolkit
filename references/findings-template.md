# FINDINGS.md — Template

The shape of an Implementation Finding. Rules this shape must satisfy live in
`references/FINDINGS-CONTRACT.md` — this file is the *shape*, not the
*rules*; do not add a checklist here (see `references/plugin-design-principles.md`
on keeping template and governance content in separate files).

Owned by `architecture-discovery`. Placed alongside the discovery report it
came from, e.g. `docs/features/<feature>/FINDINGS.md`, or `docs/FINDINGS.md`
for a whole-repository scan.

## File header

```markdown
# Implementation Findings — <scope name>

Scan source: architecture-discovery
Last scanned: <date>
```

## Per-finding entry

```markdown
### IF-001 — <short, neutral description>

- **Source:** architecture-discovery
- **Status:** Open
- **Observation:** <what was observed, stated as fact, not judgment>
- **Evidence:** <file/module path, an exact searchable pattern, or a
  reproducible command — line numbers optional, add only when they aid
  precision; never "I believe" or "likely">
- **Impact:** <plausible consequence if the observation holds — a technical,
  observable consequence, never a business/financial judgment, and never an
  instruction>
- **Confidence:** High | Medium | Low
- **Related:** <FC-/FR- ID from the PRD, ADR ID, another IF- ID, or
  `Supersedes IF-0NN` if this finding is a regression of a resolved one>
```

`Status: Accepted` adds:

```markdown
- **Reason:** <why the risk is deliberately not being fixed>
- **Accepted by:** <user, or the ADR that recorded the acceptance>
```

`Status: Closed` adds:

```markdown
- **Resolved by:** <ADR ID | commit hash | issue link>
```

## Confidence guide

- **High** — the evidence directly shows the condition (e.g. "no `catch`
  block for the specific exception in the code path that inserts this
  record").
- **Medium** — the evidence is suggestive but the surrounding context is
  incomplete (e.g. "no retry logic visible in this module; an infrastructure
  layer outside the repository could still provide it").
- **Low** — inferred from absence alone, with a plausible alternative
  explanation not ruled out.

## Example

```markdown
### IF-001 — No handling for duplicate-email insert conflict

- **Source:** architecture-discovery
- **Status:** Open
- **Observation:** The registration insert path has no handling for a
  unique-constraint violation on the email column.
- **Evidence:** `src/auth/register.ts`, registration handler — the insert
  call is unconditional; no `catch` targets Prisma error code `P2002`
  anywhere in the function, and no wrapping error handler for it exists
  upstream in the request path.
- **Impact:** Two concurrent signups with the same email may both reach the
  insert; the second fails with an unhandled exception, returned as HTTP 500
  instead of a conflict response.
- **Confidence:** High
- **Related:** FC-001
```
