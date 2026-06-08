# Project Instructions Templates

Three templates work together: a **pointer** (goes in the Claude Project instructions field, never changes), a **context file** (the living project document stored externally), and a **store index** (the registry linking all projects).

For folder structure, document types, linking conventions, and naming rules, see `knowledge-store-conventions.md`.

---

## Template 1: Project instructions pointer

Paste this into the Claude Project's **"What would you like this project to achieve?"** field. Replace the bracketed values. This pointer never needs to change after initial setup — all living context is in the external file.

---

    Before answering any question, read the store index from [store-index-uri],
    find this project's context file, read it, and treat it as the project context.

    ## Meta
    Store index: [store-index-uri]
    Project: [project-name]

---

## Template 2: Context file

Store at `[project-name]/context.md` within the knowledge store. Populate each section; omit sections that do not apply. The `## Meta` block is managed by `harvest-context` — do not edit it by hand.

---

    # [Project Name]

    ## Purpose
    [One sentence describing what this project is and why it exists.]

    ## Products and Systems
    **[Name]**: [What it does. Stage: idea / in development / in production.]

    ## Key People
    **[Name]** — [Role]

    ## Decisions
    Choices already made that should not be relitigated.

    - [Decision stated as a declarative sentence]

    ## Rules
    Business or technical constraints that govern future work.

    - [Rule stated as a declarative sentence]

    ## Definitions
    Terms that mean something specific in this context.

    **[Term]**: [Definition]

    ## Open Questions
    Unresolved issues worth tracking.

    - [Question]

    ## Anti-Patterns
    Approaches tried and abandoned.

    - [What was tried and why it was abandoned]

    ## Writing Style
    [Optional: tone, formatting preferences, phrases to avoid, standard closings]

    ## Related Docs
    [Links to ADRs, meeting notes, research, and other docs in this project space.
    Format per backend — see knowledge-store-conventions.md → Linking conventions.]

    - [ADR-001: Title](decisions/adr-001-title.md)
    - [Project index](README.md)

    ---

    ## Meta

    Storage: [uri-of-this-file]
    Accepted since last consolidation: 0
    Consolidation threshold: 12

    Sources:
    - gmail: last harvested —

---

## Template 3: Project index

Store at `[project-name]/README.md`. Created by `bootstrap-project`, updated by `harvest-context` when new linked docs are added.

---

    # [Project Name]

    [One-sentence description of the project.]

    ## Documents

    | Document | Description |
    |---|---|
    | [Context](context.md) | Living project context |
    | [Decisions](decisions/) | Architecture Decision Records |
    | [Meetings](meetings/) | Meeting notes |
    | [Research](research/) | Research and investigation notes |

    ---
    [← Store index](../README.md)

---

## Template 4: Store index

Store at the root of the knowledge store as `README.md`. Created by `bootstrap-project` on first use, updated whenever a project is added or harvested.

---

    # Knowledge Store

    | Project | Context | Status | Last harvested |
    |---|---|---|---|
    | [Project name] | [context](project-name/context.md) | active | YYYY-MM-DD |

---

## Notes

**One store per team**: all projects share a single store. The store index is the single entry point.

**Consolidation threshold**: default is 12 accepted entries. Adjust in the context file's `## Meta` block for projects that move faster or slower.

**Adding a harvest source**: add a line to `Sources:` in `## Meta`:

    Sources:
    - gmail: last harvested 2026-06-01
    - slack:#eng-backend: last harvested 2026-06-01
    - jdbc:support_tickets.resolution: last harvested 2026-05-15

**Linked docs**: when `harvest-context` creates a new ADR or meeting note, it adds a link to `## Related Docs` in context.md and a row to the project index. Both are updated automatically.
