---
description: Connect to an existing knowledge store or set up a new one. For new stores: interview the user, scaffold the store, and produce the setup document plus the Project instructions pointer. For existing stores: accept the setup document from a colleague and output the pointer only.
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

Note which are connected and which are not. You will use this inventory to populate the setup form in Step 3.

---

## Step 2: New store or existing store?

Ask the user:

> "Are you setting up a **new** knowledge store, or connecting to an **existing** one?
>
> - **New store** — I'll walk you through setup and create the store.
> - **Existing store** — Someone on your team has already set this up. Ask them for their `context-manager-setup.md` and upload it here."

**If existing store:** wait for the user to upload `context-manager-setup.md`. Once received:

1. Read the file and extract the `Store index:` URI.
2. Attempt to read the store index from that URI to confirm it resolves. If it fails, report the error and stop — the file may be stale or the MCP may not be connected.
3. Skip to Step 5. Do not run Steps 3 or 4.

**If new store:** proceed to Step 3.

---

## Step 3: Render the setup form

Render a single setup form widget in the conversation using the resource inventory from Step 1. Do not ask the setup questions conversationally — the form is the only intake interface for this step.

**Form fields:**

**Knowledge store backend** (radio group): list every Tier 2 backend. Mark connected ones as selectable; mark unconnected ones as disabled with a "not connected" label. Pre-select the recommended option (GitHub or Notion/Confluence if available; otherwise the best connected option). Include a one-line description for each backend drawn from `storage-backends.md`.

**Store root location** (text input): label "Store root path or location". Placeholder text should match the selected backend's URI format (e.g. `github:org/knowledge-base/main/README.md`, `notion:page-id`). Update the placeholder dynamically when the backend selection changes if possible; otherwise note the expected format next to the field.

**Default harvest sources** (checkbox group): list every detected Tier 1 source with checkboxes. Check all connected sources by default. Include a note below the group: "Per-project overrides are always available."

**Harvest frequency** (radio group): Weekly (default), Bi-weekly, Monthly.

**Consolidation threshold** (number input): default 12. Include a brief inline note: "Higher = less frequent cleanup passes."

**Global rules** (textarea): label "Global rules (optional)". Placeholder: `e.g. Never store PII in the knowledge store. Use metric units only.` Leave empty by default.

An **Initialize →** button at the bottom. Disable it until the backend is selected and the store root field is non-empty.

**On submit:** fire `sendPrompt()` with this structured format:

```
Context manager setup submitted.

Backend: [selected backend]
Store root: [entered value]

Default harvest sources:
  - [source]: enabled
  - [source]: disabled

Harvest frequency: [selection]
Consolidation threshold: [N]

Global rules:
  [rules text, or "None"]
```

When the `sendPrompt()` message arrives, treat it as the complete interview answers and proceed to Step 4.

**If no Tier 2 backend is connected:** render the form with all backend options disabled. Display a notice above the form explaining which MCPs to connect and where to find them in Claude.ai settings. Do not proceed until the user reconnects and re-runs the workflow.

---

## Step 4: Scaffold the knowledge store

Using the backend and root location from the submitted form, create the store index now — before generating the document.

Write the store index (Template 4 from `project-instructions-template.md`) to the store root. At this point the table is empty; projects will be added by `bootstrap-project`.

For GitHub: create the file at `[repo]/README.md`. Commit message: `chore: initialize knowledge store`.  
For Notion/Confluence: create the root page with the store index content.  
For other backends: create the file at the specified root path.

If the write fails, stop and report the error before continuing.

Record the resolved store index URI — you will need it in the document and the pointer.

---

## Step 5: Generate the setup document

Produce a complete, fully populated markdown document. No blanks, no placeholders. Every field reflects what was learned in Steps 1–4.

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

## Step 6: Output the deliverables

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

> **One manual step required:**
>
> In the Context Manager Project, go to **Project files** and upload `context-manager-setup.md`.
>
> Once that's in place, run **"bootstrap-project"** from the Context Manager Project for each project you want to register. Then run **"harvest-context"** weekly — from the Context Manager to harvest all projects at once, or from any individual project to harvest just that one.

**If connecting to an existing store** (arrived here from Step 2): skip Deliverable 1. Output only the pointer, and tell the user:

> Paste the block above into the Context Manager Project's instructions field. The store is already set up — your `context-manager-setup.md` is already in Project files. Run **"bootstrap-project"** to register new projects, or **"harvest-context"** to start harvesting.
