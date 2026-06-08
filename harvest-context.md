---
description: Scan connected sources for knowledge candidates, present for review, and write accepted entries back to the knowledge store. Runs in two modes — manager mode iterates all registered projects; project mode scopes to a single project. Creates linked docs for substantive entries. Auto-consolidates when the accepted entry count crosses the threshold.
---

## Step 1: Determine mode and locate context

Read `storage-backends.md` and `knowledge-store-conventions.md` before doing anything else.

**Detect mode from the Project instructions:**

- If the instructions contain `Store index:` but no `Project:` field, or contain `Mode: manager`: **manager mode**. The setup document is in Project files — read it from context. Proceed to the manager mode path below.
- If the instructions contain both `Store index:` and `Project:`: **project mode**. Proceed to the project mode path below.
- If neither field is present: stop.

  > "This project's instructions aren't configured for context management. Run **initialize-context-manager** to set up the Context Manager, or **bootstrap-project** to set up an individual project."

---

### Manager mode path

Read the setup document from Project files (already in context — no fetch needed). Extract the store index URI.

Read the store index. Build the list of all registered projects and their context file URIs.

Run Steps 2 through 7 for each project in sequence. Before starting each project, tell the user which project is being harvested. After all projects are complete, go to Step 8 and give a summary across all projects.

If a project's context file cannot be read, skip it, note the error, and continue to the next project.

---

### Project mode path

Read the Project instructions and extract:
- `Store index:` — URI of the store index
- `Project:` — name of this project

Read the store index from its URI. Find the row matching the Project name and extract the context file URI. Read the context file.

If any read fails, stop and report: which URI failed, which tool was used, and the specific error.

---

## Step 2: Parse the context file

Extract:

**Search terms**: product names, system names, key people, defined terms, and domain-specific vocabulary. Derive entirely from the file — do not invent terms or ask the user.

**Sources list**: from `## Meta` → `Sources:`. If absent, check the setup document (in Project files or context) for the default harvest sources and use those.

**Entry counter**: `Accepted since last consolidation`. Treat as 0 if absent.

**Backend**: infer from the `Storage:` URI scheme. Determines linking conventions and whether linked docs can be created.

**Global rules**: if a setup document is in Project files, extract any Global Rules and treat them as implicit constraints during extraction (Step 5). Do not add them as candidates — they are already in force.

---

## Step 3: Auto-consolidate if threshold is reached

If `Accepted since last consolidation` >= `Consolidation threshold`, run the consolidation pass before harvesting:

1. Review all entries across Decisions, Rules, Definitions, and Open Questions.
2. For each group: remove exact duplicates; merge near-duplicates; resolve conflicts (newer entry wins — flag any you are not confident resolving and ask the user before proceeding); promote any Open Question that now has a clear answer in a newer Decision or Rule.
3. Present the consolidated version with a summary:
   - Entry count before and after
   - What was merged or deprecated (deprecated entries stay as strikethrough — do not delete)
   - Any conflicts requiring human input
4. Ask the user to confirm before continuing.
5. Reset `Accepted since last consolidation` to 0.

In manager mode, consolidate each project independently if its threshold is reached.

---

## Step 4: Query each source

For each source in the Sources list, retrieve content since the `last harvested` date (or last 7 days if no date is set). Apply the search terms from Step 2.

**Conversational sources** (gmail, slack, outlook, teams): search for threads or messages containing the search terms. Exclude calendar events, auto-replies, and already-processed threads. For Slack, scope strictly to the specified channel.

**Database sources** (`jdbc:tablename`): query text-bearing columns filtered to rows created or modified within the harvest window.

If a listed source has no connected MCP, skip it and note the gap.

---

## Step 5: Extract candidates

Apply the context librarian extraction prompt to each retrieved item:

> You are a context librarian. Review the following content and extract items that belong in one of these four categories:
>
> - **Decision**: a choice made that should not be relitigated
> - **Rule**: a business or technical constraint that governs future work
> - **Definition**: a term that means something specific in this context
> - **Open question**: an unresolved issue worth tracking
>
> Discard pleasantries, scheduling logistics, and anything not relevant to the project. For each item include: category, one-sentence statement, source (origin + date), and confidence (high / medium / low). Return only medium or high confidence items.

Deduplicate across sources. Discard any candidate that restates a Global Rule already in the setup document.

---

## Step 6: Present candidates for review

Show candidates grouped by category. For each, display: category, statement, source, confidence.

Offer three actions:
- **Accept** — proceed to placement decision below
- **Reject** — discard
- **Refine** — ask a clarifying question, then accept the refined version

**Placement decision for accepted entries:**

For backends that support linked docs (GitHub, Notion, Confluence), ask for each accepted entry:

> "Add inline to context.md, or create a linked document?"

Suggest a linked doc when (see `knowledge-store-conventions.md` → When to create a linked doc):
- A Decision has more than one sentence of reasoning → offer to create an ADR
- A block of meeting content is worth preserving → offer to create a meeting note
- A research finding would bloat context.md → offer to create a research note

For Google Drive and file mounts, always add inline.

In manager mode, present candidates per project before moving to the next.

---

## Step 7: Write updates

Execute writes in order. Report any failure immediately.

**Write 1: Linked docs** (if any were chosen)

For each entry destined for a linked doc, populate the appropriate template from `knowledge-store-conventions.md`, write it to the correct subfolder with the correct naming convention, and record its URI.

**Write 2: context.md**

- Add inline entries to their appropriate sections.
- Add links for each new linked doc to `## Related Docs`, using the linking convention for this backend.
- Increment `Accepted since last consolidation` by the total number of accepted entries.
- Update each source's `last harvested` date to today.

Write the updated context file back to its URI.

**Write 3: Project index** (if new linked docs were created)

Open `[project-name]/README.md` and add a row for each new linked doc. Write it back.

**Write 4: Store index**

Update the `Last harvested` column for this project in the store index to today. Write it back.

If any write fails: report the error, output the full content that was meant to be written, and tell the user to update the file manually at the relevant URI.

---

## Step 8: Summary

**Project mode**: report candidates found, accepted, and rejected; which entries went inline vs. linked docs (with links to new files); updated source dates; consolidation threshold progress (warn if > 80% full).

**Manager mode**: report a table across all projects:

| Project | Candidates | Accepted | Rejected | New linked docs | Status |
|---|---|---|---|---|---|
| [name] | N | N | N | N | ✓ harvested |
| [name] | — | — | — | — | ✗ skipped ([reason]) |

Follow with any consolidations that were triggered and any write failures requiring manual intervention.
