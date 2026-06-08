# Knowledge Store Conventions

A knowledge store is a collection of interlinked markdown documents that serves as shared memory between Claude and the humans on a project. Claude reads and writes it programmatically; humans browse, search, and edit it directly in the host platform. It is not Claude's private memory — it is a first-class artifact that should be as readable and navigable by humans as by Claude.

---

## Store structure

One store per organization or team. Within the store, one folder or page space per project.

```
[store root]
├── README.md                        ← store index / registry (links to all projects)
└── [project-name]/
    ├── README.md                    ← project index (links to all docs in this project)
    ├── context.md                   ← living project context (managed by harvest-context)
    ├── decisions/
    │   └── adr-001-short-title.md
    ├── meetings/
    │   └── 2026-06-07-short-title.md
    └── research/
        └── topic-name.md
```

For wiki-style platforms (Notion, Confluence), folder hierarchy maps to page hierarchy. The store root is a top-level page or space; each project is a sub-page; decisions, meetings, and research are child pages within the project.

---

## Document types

### context.md
The living project context, managed by `harvest-context`. Contains the standard sections (Purpose, Products and Systems, Key People, Decisions, Rules, Definitions, Open Questions, Anti-Patterns, Writing Style) plus a `## Related Docs` section with links to other documents in the project space, and a `## Meta` block at the bottom.

The context file is the entry point for Claude. All other documents are linked from here.

### Architecture Decision Records
Location: `decisions/`  
Naming: `adr-NNN-short-title.md` (NNN zero-padded, e.g. `adr-001`, `adr-012`)

```markdown
# ADR-NNN: [Title]

**Date**: YYYY-MM-DD
**Status**: proposed | accepted | deprecated | superseded by ADR-NNN

## Context
What situation or problem motivated this decision?

## Decision
What was decided?

## Consequences
What becomes easier or harder as a result?
```

### Meeting notes
Location: `meetings/`  
Naming: `YYYY-MM-DD-short-title.md`

```markdown
# [YYYY-MM-DD] [Title]

**Attendees**: [names]

## Notes
[Freeform]

## Decisions made
- [Decision]

## Action items
- [ ] [Task] — [owner]
```

### Research notes
Location: `research/`  
Naming: `topic-name.md`  
Format: freeform. Summarize findings, link to sources, note conclusions.

### Project index (README.md)
One per project. Lists every document in the project space with a one-line description and a link. Created by `bootstrap-project`, updated by `harvest-context` when new linked docs are added.

### Store index (root README.md)
The registry. Lists all projects with a link to their context file, their status, and their last-harvested date. Created by `bootstrap-project` if it doesn't exist, updated whenever a new project is bootstrapped or harvested.

---

## Store index format

```markdown
# Knowledge Store

| Project | Context | Status | Last harvested |
|---|---|---|---|
| Acme Rebrand | [context](acme-rebrand/context.md) | active | 2026-06-07 |
| Q3 Migration | [context](q3-migration/context.md) | active | 2026-05-31 |
```

---

## Linking conventions

Links between documents serve two purposes: human navigation and Claude's ability to follow references during a harvest or query. Always link back to context.md from any linked doc so humans and Claude can navigate in both directions.

### GitHub (relative markdown links)

From `context.md`, link forward into the project space:
```markdown
## Related Docs
- [ADR-001: Drop legacy API](decisions/adr-001-drop-legacy-api.md)
- [Kickoff meeting](meetings/2026-06-07-kickoff.md)
- [Auth provider comparison](research/auth-provider-comparison.md)
- [Project index](README.md)
```

From any linked doc, link back:
```markdown
---
[← Back to context](../context.md)
```

From any doc, link to store index:
```markdown
[← Store index](../../README.md)
```

### Notion

The Notion MCP returns a page URL when creating a page. Store that URL as the link target in `## Related Docs`. Notion's native backlinks handle reverse navigation automatically.

```markdown
## Related Docs
- [ADR-001: Drop legacy API](https://www.notion.so/abc123...)
- [Kickoff meeting](https://www.notion.so/def456...)
```

### Confluence

Use the Confluence page URL or space-relative path:
```markdown
## Related Docs
- [ADR-001: Drop legacy API](https://wiki.example.com/spaces/PROJ/pages/123)
- [Kickoff meeting](https://wiki.example.com/spaces/PROJ/pages/456)
```

---

## When to create a linked doc vs. append inline

**Create a linked doc when:**
- A harvested Decision has enough context to warrant an ADR (more than one sentence of reasoning)
- A harvest surfaces a block of meeting content worth preserving as a standalone record
- A research thread has substance that would bloat context.md
- The entry is something a human would want to find and link to directly

**Stay inline in context.md when:**
- The entry is a crisp one-liner (most Decisions, Rules, Definitions)
- The backend doesn't support rich linking (Google Drive, file mounts)
- The project is small and navigation overhead isn't worth it

---

## Naming conventions

| Type | Pattern | Example |
|---|---|---|
| Project folder / space | lowercase-hyphens | `acme-rebrand` |
| Context file | `context.md` | `context.md` |
| Project index | `README.md` | `README.md` |
| ADR | `adr-NNN-short-title.md` | `adr-001-drop-legacy-api.md` |
| Meeting notes | `YYYY-MM-DD-short-title.md` | `2026-06-07-kickoff.md` |
| Research | `topic-name.md` | `auth-provider-comparison.md` |
| Store index | `README.md` | `README.md` |

All filenames: lowercase, hyphens for spaces, `.md` extension, no special characters.
