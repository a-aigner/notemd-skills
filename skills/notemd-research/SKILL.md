---
name: notemd-research
description: Use when navigating, searching, or reasoning over a user's notemd knowledge base via the notemd-cli / MCP server (notes + imported PDFs/sources), or when helping structure/organize a notemd project. Covers the read tools available today (search, graph, read source text, status), the Premium / app-open / .aiignore rules, and how to structure a notemd vault (folder notes, [[wikilinks]], notes vs knowledge sources, citations).
---

# notemd — driving the CLI/MCP and structuring a project

**notemd** is a macOS knowledge-base app. A user's project has two corpora:

- **Articles** (their own notes) — Markdown files in an **article vault**, organized as an Obsidian-style **folder-note hierarchy** with `[[wikilinks]]`.
- **Knowledge sources** (imported material — PDFs, web links, datasets, code repos) — stored in a **knowledge vault**; PDFs are extracted to plaintext and indexed for hybrid (semantic + keyword) search.

You reach this through **`notemd-cli`** — a headless CLI **and** an **MCP server** — which reads what the app has already indexed. This skill covers what to call and how to think about a notemd project.

---

## The golden rules (read these first)

1. **Reads launch the app; writes need it already open + Premium.** The CLI/MCP is a thin client: the notemd app runs every search, graph query and document read (it starts hidden if it isn't running — the first call can take a few seconds; don't retry while waiting). Write/create/delete operations on knowledge sources are **app-mediated** too, but they *never* launch the app: they require notemd to be running already and an active subscription. Never claim you changed the vault unless a write tool actually confirmed it.
2. **Call `get_status` / `notemd-cli status` first when in doubt.** It reports Premium entitlement, project count, whether the app bridge is available, and whether writes are currently possible. Don't guess.
3. **Premium-gated.** The CLI/MCP only works on a subscribed device. If a call says a subscription is required, tell the user once and stop — do **not** retry in a loop.
4. **`.aiignore` is access control — respect it.** Excluded documents won't appear in results and reading them is refused. If content seems missing, it may be intentionally excluded. Never try to work around it (e.g. guessing paths).
5. **Only indexed content is searchable.** Missing results can mean "not indexed yet," not "not present" — the user may need to (re)index in the app.
6. **Cite honestly.** Ground answers in the `title`/`path`/`snippet`/`sourceID` you actually retrieved. Don't fabricate sources or quote text you didn't read.
7. **No image understanding.** `get_attachments` returns *paths* to referenced images, not their contents.

---

## How you connect

- **MCP (preferred):** the host launches `notemd-cli mcp --project "<name>"`. Discover tools with `tools/list` (each tool is annotated `access: read|write` and `requiresAppOpen`); call them with `tools/call`. Each call returns `{ "content": [ { "type": "text", "text": "<JSON or text>" } ] }` — for search/graph/list tools the `text` is a JSON string to parse. A tool-level failure returns `isError: true` with a human-readable message.
- **CLI (alternative):** `notemd-cli <cmd> … --json` and parse stdout; exit code `1` = denied/not-found/error (message on stderr).
- **Choosing a project:** `--project "<name>"` (look names up with `list-projects`), or explicit `--article-vault PATH --knowledge-vault PATH`.

---

## Current capabilities (what exists today)

Most tools are **reads** (the app answers them, launching itself if needed; `list_source_anchors`, `export_bibtex`, `job_status`, `list_projects`, `get_status` work even with the app closed). The full set of **writes** to knowledge sources (sources, folders, anchors) is live — these are **app-mediated**: notemd must be open (else you get `app_not_running`) and the subscription must be active. Notes/articles are written a different way — directly on the filesystem (see *Editing notes*), not through a write tool.

### Knowledge-source write tools (app-mediated — require notemd open + Premium)
These mutate **knowledge sources** (PDFs/refs). Notes/articles are written a different way — via the filesystem (see *Editing notes* below), not these tools.

| Tool | Use |
|---|---|
| `add_source_pdf(path, folderID?)` | Import a local PDF as a source. Returns the `sourceID`; extraction runs in the background (read later with `get_source_text`). |
| `add_source_ref(kind, title, url)` | Add a non-file reference; `kind` ∈ `webLink \| dataset \| codeRepository \| note`. `url` required. |
| `edit_source_metadata(sourceID, fields)` | Update citation fields (`title, authors, year, venue, publisher, doi, url`). |
| `set_source_tags(sourceID, add?, remove?)` | Add/remove `SourceTag` raw values (controlled vocab: `background`, `theoretical_foundation`, `method_precedent`, `empirical_support`, `dataset_source`, `counterevidence`, `review_meta_analysis`, `definition_source`). Unknown values are ignored and returned in `rejected`. |
| `move_source(sourceID, folderID?)` | Move into a folder (omit `folderID` for the project root). |
| `delete_source(sourceID)` | **Async**: returns a `jobID`; poll `job_status(jobID)`. |
| `create_folder` / `rename_folder` / `delete_folder` | Knowledge folders (`delete_folder` needs the folder empty). |
| `create_anchor` / `update_anchor` / `delete_anchor` | Anchors (highlights/notes) on a source. |

**Write rules:** call `get_status` first — if `writesAvailable` is false, tell the user to open notemd (and/or that Premium is required) and stop; **don't retry-loop** on `app_not_running` or `not_entitled`. Writes to a source excluded by `.aiignore` are refused. CLI equivalents mirror these: `source add-pdf|add-ref|edit|tag|move|delete`, `folder create|rename|delete`, `anchor create|update|delete`, `job status` (all take `--project`).

### Read tools (10)
| Tool | Use |
|---|---|
| `get_status()` | Premium / app-bridge / writes-available snapshot. Call first if a write might be needed. |
| `hybrid_search(query, scope?, limit?)` | **Primary retrieval.** Semantic + keyword over the project. `scope` ∈ `articles \| knowledge \| sources \| both` (default `both`; `sources` = alias for `knowledge`). Returns `SearchResult[]`: `{ title, path, snippet, score, kind, sourceID }`. `kind` ∈ `note \| pdf \| webLink \| dataset \| codeRepository`. |
| `get_source_text(sourceID)` | **Read a knowledge source (PDF etc.) as full text**, by the `sourceID` from a search result. Use this — **not `get_document`** — for PDFs/sources (`get_document` reads raw bytes and returns empty for a binary PDF). |
| `list_source_anchors(sourceID)` | List a source's **bookmarks** (page/text anchors): each `{ id, kind, title, page, quote, note, cite }`. Use it to **discover anchor IDs** and copy the ready-made `cite` link when citing a bookmark from a note (see *Citing a source*). |
| `get_document(path)` | Full text of a **note** by its vault-relative `path`. Refused if `.aiignore`-excluded or a dotfile. |
| `graph_related(node, limit?)` | Semantically-similar notes (nearest neighbors in embedding space) of a note **title**. Good for "what else is about this?" |
| `graph_links(node, direction?, hops?)` | Notes connected to `node` (a title) by `[[wikilinks]]` — the user's explicit structure. `direction` ∈ `in \| out \| both`. |
| `list_documents(scope?)` | Enumerate documents with vault-relative paths. Use to orient or resolve a title→path. |
| `list_projects()` | Names + vault/index paths in the registry. |
| `get_attachments(document)` | Image/attachment **paths** a document references (`![[embeds]]`, `![](images)`). Paths only, no image content. |
| `export_bibtex(sourceIDs?)` | Render the project's sources (or a subset) as BibTeX from their citation metadata. |
| `job_status(jobID)` | Status of an async job (e.g. from `delete_source`): `processing \| completed \| failed`. |

### CLI subcommands (mirror the read tools)
`notemd-cli list-projects` · `search "<query>" [--scope …] [--limit N]` · `graph links <title> [--direction …] [--hops N]` · `graph related <title> [--limit N]` · `source text <sourceID>` · `source anchors <sourceID> [--json]` · `status` · `mcp` — all accept `--project "<name>"` (or explicit vault paths) and `--json`.

---

## Recommended workflows

- **Answer a question from the knowledge base:**
  1. `hybrid_search(query, scope: "both", limit: 5–10)`.
  2. Read the top snippets; for the 1–3 most relevant, get the full text — `get_source_text(sourceID)` for a source/PDF, `get_document(path)` for a note.
  3. Answer, citing `title`/`path`. If nothing relevant returns, say the base doesn't cover it rather than inventing.
- **"What has the user written about X?"** → `scope: "articles"`. **"What do my sources say about X?"** → `scope: "knowledge"`.
- **Explore around a note:** `graph_related` (similar topics) + `graph_links` (their explicit links), then read the interesting ones.
- **Read a PDF end-to-end:** `hybrid_search` to find it → take its `sourceID` → `get_source_text(sourceID)`.

---

## How a notemd project is structured (guidance for organizing a vault)

Understand the model before suggesting or making organizational changes:

- **On-disk is the source of truth.** Articles are `{Title}.md` files with front matter; the article vault is a **folder-note hierarchy** (a folder can have a companion note). Links between notes are Obsidian **`[[wikilinks]]`**. (The app also shows in-memory `notemd://` links, but those are **not** an on-disk format — anything you write to disk uses `[[wikilinks]]` + folder notes, never a `parent_id` field.)
- **Notes vs knowledge sources — pick the right home:**
  - A **note/article** is the user's *own* thinking: summaries, claims, connections, an outline. Notes are where synthesis lives and where wikilinks/citations connect ideas.
  - A **knowledge source** is *imported external* material (a PDF, a web page, a dataset, a repo). Sources are evidence you cite; you don't rewrite them.
  - Rule of thumb: if the user is *authoring* it, it's a note; if they're *referencing* it, it's a source.
- **Folders group by topic**, and a folder note gives that topic an overview/index. Keep hierarchies shallow and topical rather than deep and rigid.
- **Wikilinks encode the user's intended relationships** (`graph_links`); **semantic similarity** surfaces implicit ones (`graph_related`). Prefer explicit `[[wikilinks]]` when a real conceptual connection exists.
- **Citations connect a note to a source** — a note makes a claim and cites the source that supports it, so evidence is traceable back to `sourceID`/path.
- **Tags** are lightweight cross-cutting labels; use them for themes that cut across folders, not as a replacement for folder structure.
- **Canonical workflow for building knowledge:** *import sources → let the app extract/index them → search & traverse the graph to find what's relevant → write notes that synthesize and cite those sources → connect notes with `[[wikilinks]]`.*

---

## Editing notes/articles — do it directly on the filesystem

**Notes are the disk-mirrored half of notemd**, so you don't need any CLI/MCP write tool for them: **create, edit, move, and delete the `.md` files in the article vault directly** (a filesystem connector / file tools), and notemd reconciles them into its graph + sidebar. This is by design — the app watches the vault and merges on-disk changes.

- **Where:** the **article vault path** is `articleVaultPath` from `list_projects`. Notes are `.md` files; subfolders are the topic hierarchy.
- **Create a note:** write a new `Some Title.md` (in the root, or in a subfolder for its topic). **Plain Markdown is enough** — start with a `# Title` and the body. notemd assigns the note's identity and normalizes it on merge; you do **not** need to write front matter or a UUID.
- **Link notes:** use `[[Note Title]]` (or `[[Note Title|alias]]`). notemd resolves these to real links on merge — this is how you build the graph.
- **Cite a source or bookmark in a note:** note→source citations use inline Markdown links to a `notemd://cite/…` URL (unlike note→note `[[wikilinks]]`), and they round-trip verbatim on disk. Two forms:
  - **Whole source:** `[label](notemd://cite/<sourceID>)`.
  - **A specific bookmark (page/text anchor):** `[label](notemd://cite/<sourceID>?anchor=<anchorID>&page=N)`. Discover the `sourceID` from a search result and the `anchorID` (+ a ready-made `cite` string) from **`list_source_anchors(sourceID)`** — paste its `cite` value straight into the link target. To bookmark a passage yourself first, use the `create_anchor` write tool (it returns the new `anchorID`).
- **Edit a note:** modify the file. If it has a `--- … ---` front-matter block at the top, **leave that block intact** (it carries the note's id) and edit the body below it. (Even if the block is lost, notemd restores the id by file path — but don't rely on it.)
- **Rename / move / organize:** rename the file (and/or the `# Title`), or move it into a subfolder — that's its place in the hierarchy. Create folders as directories. Identity is preserved.
- **Delete:** delete the file. (notemd may defer removing *many* notes at once as a safety measure — delete a few at a time.)

**Filesystem caveats:**
1. Changes reconcile **when notemd is running with that project open** (same "app must be active" constraint as source writes). Tell the user to open the project if they need it live.
2. **Never hand-write a broken front-matter block.** A malformed `--- … ---` YAML block makes notemd *skip* the file. Prefer plain Markdown (no front-matter block) or leave an existing valid block untouched.
3. **`.aiignore` does not restrict raw filesystem access** — it only gates the CLI/MCP tools. If the user has excluded notes, respect that intent even though the files are reachable.
4. Give new notes distinct titles/filenames; don't duplicate an existing note's id.

---

## Honesty & limits

- You can **read** freely; **write to knowledge sources** via the app-mediated tools (add/edit/tag/move/delete sources, folders, anchors) when notemd is open + subscribed; and **write notes** by editing the vault's `.md` files directly (see *Editing notes*). Only claim a write succeeded if the tool returned success (for sources) or the file was actually written (for notes) — on `app_not_running`/`not_entitled`, report that plainly and don't retry-loop.
- Respect `.aiignore`, Premium gating, and "only indexed content is searchable." Surface those constraints to the user plainly rather than working around them — including for direct file edits, where `.aiignore` isn't enforced but the user's intent still holds.
