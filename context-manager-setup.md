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

Backend: Notion
Store index: notion:37999e1b-498a-8113-b062-c403d6572565
Initialized: 2026-06-08

---

## Default Harvest Sources (Tier 1)

Sources harvested by default for every project unless overridden in the
project's context file.

- gmail: enabled
- google-calendar: enabled
- gdrive: enabled (docs created or modified since last harvest only; store subtree excluded)
- notion: disabled (enable to harvest Notion pages outside the knowledge store; store subtree excluded automatically)

---

## Default Settings

Harvest frequency: weekly
Consolidation threshold: 12

---

## Global Rules

None

---

## Available Backends

Backends detected at initialization time. Add new ones by connecting the
relevant MCP in Claude.ai and updating this section.

**Tier 1 (raw sources)**
- Gmail ✓
- Google Calendar ✓
- Google Drive ✓ (as harvest source; store subtree excluded when also Tier 2)
- Notion ✓ (as harvest source; store subtree excluded when also Tier 2)
- Slack ✗
- Outlook / Teams ✗

**Tier 2 (knowledge store)**
- Notion ✓
- Google Drive ✓
- GitHub ✗
- Confluence ✗
- SharePoint ✗
