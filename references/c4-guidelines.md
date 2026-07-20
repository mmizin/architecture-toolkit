# C4 Model — Guidelines & Conventions

The C4 model describes software architecture at four zoom levels. Use only the
levels your audience needs; the Context and Container levels cover most needs.

## Levels

1. **System Context** — the system as a single box, its users (people), and the
   external systems it interacts with. Audience: everyone.
2. **Container** — deployable/runnable units (web app, API service, database,
   message broker) and how they communicate. Audience: technical stakeholders.
   *Usually the highest-value diagram.*
3. **Component** — components within a single container. Use only where it earns
   its maintenance cost.
4. **Code** — classes/schema. Rarely worth drawing by hand; generate if needed.

## Conventions

- **One level per diagram.** Never mix Context and Container elements.
- **Label every relationship** with intent *and* technology, e.g.
  `Reads/writes → [SQL/TCP]`, `Sends events → [Kafka]`.
- Distinguish **people**, the **system in scope**, and **external systems**
  visually (color/shape).
- Keep element names and responsibilities short; detail belongs in arc42.
- Prefer **text-based** diagrams (Mermaid or Structurizr DSL) checked into the
  repo so they version alongside code.

## Mermaid — System Context example

```mermaid
C4Context
    title System Context — Example System
    Person(user, "User", "Uses the system")
    System(sys, "Example System", "Does the thing")
    System_Ext(email, "Email Provider", "Sends transactional email")
    Rel(user, sys, "Uses", "HTTPS")
    Rel(sys, email, "Sends email via", "SMTP")
```

## Mermaid — Container example

```mermaid
C4Container
    title Container — Example System
    Person(user, "User")
    Container(web, "Web App", "Next.js", "UI")
    Container(api, "API", "NestJS", "Business logic")
    ContainerDb(db, "Database", "PostgreSQL", "Persistent state")
    Rel(user, web, "Uses", "HTTPS")
    Rel(web, api, "Calls", "JSON/HTTPS")
    Rel(api, db, "Reads/writes", "SQL")
```

## Review checklist

- Right abstraction level for the audience?
- Every relationship labelled with intent + technology?
- No mixed levels in a single diagram?
- Element count manageable (split if it becomes unreadable)?
- Diagram matches the real system (cross-check with discovery)?