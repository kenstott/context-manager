---
description: Interview the user, scaffold the knowledge store, and produce the Context Manager setup document plus the Project instructions pointer. The setup document is uploaded to the Context Manager Claude Project's files; the pointer goes in its instructions field.
---

## Step 1: Inventory available resources

Read `storage-backends.md` before doing anything else.

Check which MCPs are currently connected and classify them into two tiers:

**Tier 1 — Raw sources** (information originates here; Claude harvests from these):
- Gmail
- Slack (note any configured channels)
- Outlook / Teams
- Database connections (jdbc or similar)
- Any other conversational or record-keeping MCPs

**Tier 2 — Knowledge store backends** (curated insights live here; Claude reads and writes these):
- GitHub
- Notion
- Confluence
- Google Drive
- SharePoint
- Local file mount (Claude Desktop only)

Note which are connected and which are not. You will present this inventory to the user during the interview.

---

## Step 2: Interview

Ask the following questions one at a time. Wait for each answer before continuing.

**Question 1**
Present the detected Tier 2 backends. Ask which one they want to use for the knowledge store. If more than one is connected, ask which they prefer — recommend GitHub or Notion/Confluence for their native markdown support and human browsability. If none are connected that support rich linking, note the limitation and let them choose from what is available.

**Question 2**
Where within that backend should the store root live? Ask for a path or location — for example, a GitHub repo name, a Notion parent page, a Google Drive folder path. Suggest a dedicated location rather than mixing with other files (e.g. a repo named `knowledge-base`, a Notion page named "Knowledge Store").

**Question 3**
Present the detected Tier 1 sources. Which ones should be harvested by default across all projects? They can always add or remove sources per project later — this sets the starting default.

**Question 4**
How often should harvests run? Weekly is the default. Any reason to go more or less frequent?

**Question 5**
What consolidation threshold — the number of accepted harvest entries that triggers a cleanup pass? Default is 12. Higher means less frequent consolidation; lower means tighter upkeep.

**Question 6**
Are there any rules that apply globally — across every project in this store? Things like security constraints, compliance requirements, preferred tools, communication norms. These will appear in a Global Rules section that every project inherits.

---

## Step 3: Scaffold the knowledge store

Using the backend and root location from the interview, create the store index now — before generating the document.

Write the store index (Template 4 from `project-instructions-template.md`) to the store root. At this point the table is empty; projects will be added by `bootstrap-project`.

For GitHub: create the file at `[repo]/README.md`. Commit message: `chore: initialize knowledge store`.  
For Notion/Confluence: create the root page with the store index content.  
For other backends: create the file at the specified root path.

If the write fails, stop and report the error before continuing.

Record the resolved store index URI — you will need it in the document and the pointer.

---

## Step 4: Generate the setup document

Produce a complete, fully populated markdown document. No blanks, no placeholders. Every field reflects what was learned in Steps 1–3.

The document should contain:

---

**`context-manager-setup.md`**

```markdown
# Context Manager Setup

This document configures Claude's context management system. It lives in the
Context Manager Claude Project's files and is read automatically on every turn.

---

## Architecture

Context management works in two tiers:

**Tier 1 — Raw sources**: where information originates. Claude queries these
during harvest (Gmail, Slack, databases, etc.) but never stores raw content.
These are read-only from the knowledge store's perspective.

**Tier 2 — Knowledge store**: where curated insights live. Claude reads this
at the start of every conversation and writes to it during harvests and
bootstraps. Stored as interlinked markdown — browsable by humans, managed
by Claude.

`harvest-context` is the bridge: it reads Tier 1, extracts signal, and
writes structured entries to Tier 2.

---

## Knowledge Store

Backend: [chosen backend]
Store index: [store-index-uri]
Initialized: [date]

---

## Default Harvest Sources (Tier 1)

Sources harvested by default for every project unless overridden in the
project's context file.

[List each source with its name and any scope (e.g. slack channel)]

- [source]: enabled
- [source]: enabled

---

## Default Settings

Harvest frequency: [weekly / other]
Consolidation threshold: [N]

---

## Global Rules

Rules that apply across every project in this store. Inherited automatically —
individual projects do not need to restate them.

[List rules, or "None" if the user had none]

---

## Available Backends

Backends detected at initialization time. Add new ones by connecting the
relevant MCP in Claude.ai and updating this section.

**Tier 1 (raw sources)**
[List each with connected ✓ or not connected ✗]

**Tier 2 (knowledge store)**
[List each with connected ✓ or not connected ✗]
```

---

## Step 5: Output the deliverables

Present two things to the user.

**Deliverable 1: The setup document**

Output `context-manager-setup.md` as a downloadable file.

**Deliverable 2: The Project instructions pointer**

Output the following block for the user to paste into the Context Manager Claude Project's instructions field:

    You are the context manager for a knowledge store.

    At the start of every conversation, read context-manager-setup.md from your
    project files. It defines the store location, connected sources, and global
    rules. If it is not present, you are ready to be initialized.

    When a command is given, read the matching workflow document from your project files:
    - "Initialize context manager" → initialize-context-manager.md
    - "Create topic" → bootstrap-project.md
    - "Harvest ideas" → harvest-context.md

    Read the store index and individual topic files only when the task requires it.

Then tell the user:

**If this is a first-time setup:**

> **Two manual steps required:**
>
> 1. Go to claude.ai and create a new Project named **Context Manager**. In the **"What would you like this project to achieve?"** field, paste the pointer above.
>
> 2. In that same Project, go to **Project files** and upload `context-manager-setup.md`.
>
> Once both are in place, run **"bootstrap-project"** from the Context Manager Project for each project you want to register. Then run **"harvest-context"** weekly — from the Context Manager to harvest all projects at once, or from any individual project to harvest just that one.

**If this is a re-initialization (the Context Manager Project already exists):**

> **One manual step required:**
>
> Go to the **Context Manager** Project on claude.ai, open **Project files**, and replace the existing `context-manager-setup.md` with the new one. The Project instructions pointer does not need to change.
