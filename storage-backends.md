# Storage Backends

Defines the URI schemes used across the context management workflow and how Claude resolves each to a concrete tool or API call. Every read and write operation in `bootstrap-project` and `harvest-context` routes through this reference.

For folder structure, document types, linking conventions, and naming rules, see `knowledge-store-conventions.md`.

---

## URI Schemes

### GitHub

```
github:[owner]/[repo]/[branch]/[path]
```

Example: `github:acme/knowledge-base/main/acme-rebrand/context.md`  
Store index example: `github:acme/knowledge-base/main/README.md`

| Operation | Tool | Method |
|---|---|---|
| Read file | GitHub MCP | `get_file_contents` |
| Write new file | GitHub MCP | `create_or_update_file` |
| Update existing file | GitHub MCP | `create_or_update_file` (requires current SHA) |
| Create folder | GitHub MCP | Create a `.gitkeep` file at `[folder]/.gitkeep` |
| List folder | GitHub MCP | `list_directory_contents` |

**Commit messages**: use a consistent format so harvest runs are identifiable in git history:
- Bootstrap: `chore: bootstrap [project-name] context`
- Harvest: `chore: harvest [project-name] — N entries accepted`
- New linked doc: `docs: add [doc-type] [title]`

**Branch strategy**: write directly to the default branch (`main`). If the organization prefers review workflows, write to a feature branch and note the PR URL for the user.

**Repo structure**: a dedicated repo (e.g. `org/knowledge-base`) is recommended. You can also use a `docs/` or `.claude/` subfolder within an existing repo — adjust the path prefix in the URI accordingly.

**Relative links**: use relative markdown paths within the store. See `knowledge-store-conventions.md` → Linking conventions → GitHub.

Requirements: GitHub MCP connected in Claude.ai.

---

### Notion

```
notion:[page-id]
```

Example: `notion:abc123def456ghi789`  
Store index example: `notion:root-registry-page-id`

| Operation | Tool | Method |
|---|---|---|
| Read page | Notion MCP | `retrieve_page` |
| Create page | Notion MCP | `create_page` (returns new page ID and URL) |
| Update page | Notion MCP | `update_page` |
| List child pages | Notion MCP | `list_child_pages` |
| Append block | Notion MCP | `append_block_children` |

**Page hierarchy**: the store index is a top-level Notion page. Each project is a child page of the index. Within each project page: Decisions, Meetings, and Research are child pages. Context lives in the project page body or as a pinned child page named "Context".

**Page IDs vs URLs**: when `create_page` returns a new page, store its ID (not the full URL) as the URI. The Notion MCP accepts IDs directly. Full URLs work too but are longer.

**Linking**: Notion pages link to each other by URL. When creating a linked doc, store the returned page URL in the `## Related Docs` section of context.md. Notion's native backlinks surface reverse references automatically.

**Databases**: optionally, use a Notion database for the store index instead of a plain page. This enables filtered views (by status, owner, last harvested). If using a database, each row is a project; the context page is a linked sub-page on the row.

Requirements: Notion MCP connected in Claude.ai.

---

### Confluence

```
confluence:[space-key]/[page-title-slug]
```

Example: `confluence:PROJ/Acme-Rebrand-Context`  
Store index example: `confluence:PROJ/Knowledge-Store-Index`

| Operation | Tool | Method |
|---|---|---|
| Read page | Confluence MCP | `get_page` |
| Create page | Confluence MCP | `create_page` |
| Update page | Confluence MCP | `update_page` (requires current version number) |
| List child pages | Confluence MCP | `get_child_pages` |

**Space structure**: one Confluence space for the knowledge store. Each project is a top-level page in that space. Decisions, Meetings, and Research are child pages within each project page.

**Page titles**: Confluence page titles must be unique within a space. Use the pattern `[Project Name] — [Doc Type] — [Title]` for linked docs: e.g. `Acme Rebrand — ADR-001 — Drop Legacy API`.

**Version numbers**: Confluence requires the current version number to update a page. Always read the page first to get the current version before writing.

**Linking**: use full Confluence page URLs in `## Related Docs`. Confluence's built-in "Links" panel surfaces backlinks.

Requirements: Confluence MCP connected in Claude.ai. The target space must already exist.

---

### Google Drive

```
gdrive:[path]
```

Example: `gdrive:My Drive/knowledge-base/acme-rebrand/context.md`

| Operation | Tool | Method |
|---|---|---|
| Read | Google Drive MCP | `read_file_content` or `google_drive_fetch` |
| Write (new) | Google Drive MCP | `create_file` |
| Write (update) | Google Drive MCP | `update_file` |
| List folder | Google Drive MCP | `search_files` with parent folder ID |

**Limitations**: Google Drive does not render relative markdown links between files in a navigable way. Use it for storage, but don't rely on cross-document linking. The `## Related Docs` section can still list file names for human reference — they just won't be clickable links from within Drive.

Requirements: Google Drive MCP connected in Claude.ai.

---

### Local file mount

```
file:[/absolute/path]
```

Example: `file:/mnt/projects/knowledge-base/acme-rebrand/context.md`

| Operation | Tool | Method |
|---|---|---|
| Read | `bash_tool` | `cat [path]` |
| Write | `bash_tool` | `tee [path]` or heredoc |
| Create folder | `bash_tool` | `mkdir -p [path]` |
| List folder | `bash_tool` | `ls [path]` |

**Persistent mounts only**: this scheme is only suitable for Claude Desktop with a stable filesystem mount. The `/home/claude` container resets between sessions — do not use `file:` URIs pointing there.

**Linking**: relative markdown links work if the user opens files in a markdown editor that resolves relative paths (VS Code, Obsidian, etc.).

Requirements: Claude Desktop with a persistent mount at the specified path.

---

### SharePoint

```
sharepoint:[site]/[library]/[path]
```

Example: `sharepoint:team-site/Documents/knowledge-base/acme-rebrand/context.md`

| Operation | Tool | Method |
|---|---|---|
| Read | SharePoint MCP | file read tool |
| Write | SharePoint MCP | file write tool |

Use `tool_search` to discover exact tool names for the connected SharePoint MCP — implementations vary.

Requirements: SharePoint MCP connected in Claude.ai.

---

## Resolving a URI at runtime

1. Read the scheme prefix (everything before the first `:`).
2. Look up the scheme in this document.
3. Verify the required MCP or tool is available. If not, stop and report: which URI failed, which tool is needed, and how to connect it.
4. Execute the read or write using the specified method.

Never silently fall back to a different backend. If a URI cannot be resolved, surface the error.

---

## Write failure fallback

If a write fails after a harvest or bootstrap:

1. Report the error and the URI.
2. Output the full content that was meant to be written.
3. Tell the user to update the file manually at that URI.

---

## Adding a new backend

1. Choose an unambiguous URI scheme prefix.
2. Document read and write operations with specific tool calls.
3. Note requirements, limitations, and linking behavior.
4. Add it to this file and cross-reference it in `bootstrap-project.md` and `harvest-context.md`.
