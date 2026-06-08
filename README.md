# Context Manager

A personal knowledge system built on interlinked markdown documents. Claude harvests signal from your email, Slack, and other sources, distills it into a structured knowledge store you can browse like a wiki, and carries that context into every conversation.

---

## How it works

Two tiers:

**Raw sources** — Gmail, Slack, databases. Where information originates. Claude reads these periodically but never stores raw content.

**Knowledge store** — GitHub, Notion, Confluence, or Google Drive. Where curated insights live as linked markdown files. Readable by both Claude and humans.

`Harvest ideas` is the bridge: it reads your raw sources, extracts what matters, and asks you to review before writing anything to the store.

---

## Prerequisites

- At least one knowledge store backend connected: GitHub, Notion, Confluence, or Google Drive
- At least one harvest source connected: Gmail, Slack, Outlook, or similar

This system works on the free Claude.ai plan. Heavy use (long harvest sessions, many projects) may bump into daily message limits — upgrade to Pro if that becomes a regular friction point.

---

## Setup (one time, ~10 minutes)

### Step 1 — Create the Context Manager project

Go to [claude.ai](https://claude.ai) → Projects → New Project. Name it **Context Manager**.

In the **"What would you like this project to achieve?"** field, paste this exactly:

```
You are the context manager for a knowledge store.

At the start of every conversation, read context-manager-setup.md from your
project files. It defines the store location, connected sources, and global
rules. If it is not present, you are ready to be initialized.

When a command is given, read the matching workflow document from your project files:
- "Initialize context manager" → initialize-context-manager.md
- "Create topic" → bootstrap-project.md
- "Harvest ideas" → harvest-context.md

Read the store index and individual topic files only when the task requires it.
```

### Step 2 — Upload the workflow files

In the Context Manager project, go to **Project files** and upload all six of these files:

- `initialize-context-manager.md`
- `bootstrap-project.md`
- `harvest-context.md`
- `knowledge-store-conventions.md`
- `storage-backends.md`
- `project-instructions-template.md`

### Step 3 — Initialize

Open a conversation in the Context Manager project and type:

> Initialize context manager

Claude will ask you about six questions — which backend to use for the knowledge store, which sources to harvest from, and a few preferences. It then scaffolds your store and produces a `context-manager-setup.md` file.

### Step 4 — Upload the setup file

Download `context-manager-setup.md` from the conversation and upload it to the Context Manager project's **Project files** alongside the workflow files from Step 2.

Setup is complete.

---

## Creating your first project

In the Context Manager project, type:

> Create topic

Claude interviews you about the project — what it is, who's involved, decisions already made, open questions. It then creates a structured context file and project space in your knowledge store and gives you a pointer to paste into a new Claude Project for that topic.

---

## Keeping it current

Once a week, type:

> Harvest ideas

Claude scans your connected sources (email, Slack, etc.) for anything worth adding to the knowledge store, surfaces candidates grouped by type, and asks you to accept, reject, or refine each one before writing anything. Substantive entries — decisions with real context, meeting records, research threads — get their own linked documents. Everything else gets added inline.

Run this from the **Context Manager project** to harvest all topics at once. Run it from any **individual topic project** to harvest just that one.

---

## What's in the box

| File | Purpose |
|---|---|
| `initialize-context-manager.md` | First-time setup workflow |
| `bootstrap-project.md` | Adds a new topic/project to the knowledge store |
| `harvest-context.md` | Weekly harvest workflow |
| `knowledge-store-conventions.md` | Folder structure, document types, linking conventions |
| `storage-backends.md` | URI schemes and tool mappings per backend |
| `project-instructions-template.md` | Templates for pointers, context files, and indexes |

The workflow files (`initialize`, `bootstrap`, `harvest`) are instructions Claude follows when you run a command. The reference files (`conventions`, `backends`, `templates`) are what those workflows read to stay consistent. You should not need to edit any of them.

---

## Troubleshooting

**"This project's instructions aren't configured for context management."**  
The Project instructions field wasn't set up correctly in Step 1. Re-paste the block from Step 1 into the project settings.

**"The store index URI cannot be resolved."**  
The MCP for your chosen backend isn't connected, or the path is wrong. Check [claude.ai settings](https://claude.ai/settings) → Integrations to reconnect.

**Claude isn't picking up the workflow files.**  
Make sure all six files from Step 2 are uploaded to Project files, not just attached to a conversation.

**The setup file is missing after re-opening the project.**  
`context-manager-setup.md` needs to be in Project files (permanent), not a conversation attachment (temporary). Re-upload it to the project.
