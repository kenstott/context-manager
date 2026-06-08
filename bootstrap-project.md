---
description: Interview the user, produce a populated context file, scaffold the project space in the knowledge store, register the project, and output the Project instructions pointer.
---

## Step 1: Locate or initialize the knowledge store

Read `storage-backends.md` and `knowledge-store-conventions.md` before doing anything else.

Check the current Project instructions for a `Store index:` field.

**Found**: read the store index from that URI. Note its location — you will update it in Step 5.

**Not found**: ask the user:

> "Where should the knowledge store live? This is a one-time setup. Once created, all future projects register here automatically."

Show which backends are currently available (check connected MCPs) with their URI format from `storage-backends.md`. Let them choose. If the store index does not yet exist at the chosen URI, you will create it in Step 5.

If the store index URI cannot be resolved (MCP not connected, path not found, permission error), stop and report the specific error. Do not proceed until resolved.

---

## Step 2: Interview

Ask the following questions one at a time. Wait for each answer before continuing.

**Question 1**
What is this project in one sentence?

**Question 2**
What products, systems, or codebases are involved? List names only.

**Question 3**
For each thing named: what does it do in one sentence, and what stage is it at — idea, in development, or in production?

**Question 4**
Who are the key people involved and what is each person's role? Include yourself.

**Question 5**
What decisions have already been made that should not be revisited? Conclusions only — not the reasoning.

**Question 6**
What business or technical rules apply? Constraints and non-negotiables.

**Question 7**
Are there terms that mean something specific in this context?

**Question 8**
What open questions are you carrying right now?

**Question 9** *(ask only if anti-patterns came up naturally earlier)*
Are there approaches that have been tried and abandoned?

---

## Step 3: Produce the context file

Using Template 2 from `project-instructions-template.md`, populate each section. Omit sections where the user had nothing to say.

Derive the project folder name from the project name: lowercase, hyphens for spaces. Example: `acme-rebrand`.

In `## Related Docs`, add a link to the project index (`README.md`) as the only initial entry — other links will be added as documents are created.

In `## Meta`:
- Set `Storage:` to the context file URI (e.g. `github:org/knowledge-base/main/acme-rebrand/context.md`)
- Set `Accepted since last consolidation: 0`
- Set `Consolidation threshold: 12`
- Set `Sources:` to `- gmail: last harvested —`

---

## Step 4: Confirm storage location

Ask: "Where in the knowledge store should I put this project?"

Suggest `[store-root]/[project-name]/` as the natural location. Confirm the full URI for the context file before writing. If the user wants a different backend from the store index, check that it is available and note that cross-backend linking will be limited.

---

## Step 5: Scaffold the project space and update the store

Execute these writes in order. Report any failure immediately and stop.

**Write 1: Context file**

Write the populated context file to `[project-name]/context.md` within the store. Confirm success.

**Write 2: Project index**

Using Template 3 from `project-instructions-template.md`, create `[project-name]/README.md`. Adjust links to match the backend's linking convention (see `knowledge-store-conventions.md`). Confirm success.

**Write 3: Scaffold subdirectories**

For backends that require a file to create a folder (GitHub), create placeholder files:
- `[project-name]/decisions/.gitkeep`
- `[project-name]/meetings/.gitkeep`
- `[project-name]/research/.gitkeep`

For wiki platforms (Notion, Confluence), create child pages:
- "Decisions" — child of the project page
- "Meetings" — child of the project page
- "Research" — child of the project page

For file mounts, create empty folders: `mkdir -p decisions meetings research` within the project path.

For Google Drive, skip this step — folder creation happens on first file write.

**Write 4: Store index**

Add a row to the store index for the new project:

    | [Project name] | [context-link] | active | — |

Format the context link per the backend's linking convention. If the store index does not yet exist, create it using Template 4 from `project-instructions-template.md` with this project as the first row. Confirm success.

---

## Step 6: Output the pointer

Present the following for the user to paste into the Claude Project instructions field:

    Before answering any question, read the store index from [store-index-uri],
    find this project's context file, read it, and treat it as the project context.

    ## Meta
    Store index: [store-index-uri]
    Project: [project-name]

Then tell the user:

> **Manual step required**: Go to claude.ai and create a new Project named **[project-name]** (if it doesn't exist yet). Paste the block above into the **"What would you like this project to achieve?"** field. That pointer never needs to change.
>
> Your knowledge store now has a browsable project space at `[project-folder-uri]`. You can add documents there directly or let `harvest-context` grow it over time.
>
> Run **"harvest-context"** after your first week of activity — it will pick up where bootstrap left off.

---

## Step 7: Clarify what the dedicated Claude Project is for

After outputting the pointer, make sure the user understands that creating a dedicated Claude Project for this topic is **optional**, and explain the distinction:

> The knowledge store entry you just created is what matters for harvesting. Harvests run in two modes: from the **Context Manager** project, manager mode iterates every project registered in the store index — so this project will be harvested whether or not a dedicated Claude Project for it ever exists. A separate Claude Project is **not** required for harvesting.
>
> What a dedicated Claude Project for this topic *does* buy you is a workspace where this context loads automatically on every turn, so you can ask questions, draft content, and build plans about the topic without re-establishing context each time. That is the pointer's entire job — it tells Claude to read the store index and this project's context before answering.
>
> So:
> - **Want to harvest only?** Skip the dedicated project. Run `harvest-context` from the Context Manager and it covers this project automatically.
> - **Want a thinking/planning space for this topic?** Create the Claude Project and paste the pointer. Anything you ask there is then grounded in the context file.
>
> The two are not mutually exclusive, and neither blocks the other. The store entry exists and is harvestable as soon as Step 5 completes.
