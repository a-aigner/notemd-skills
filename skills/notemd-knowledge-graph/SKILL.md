---
name: notemd-knowledge-graph
description: Use when the user asks about notemd's Knowledge Map (graph view), knowledge graph, backlinks, connections between notes, or "argument maps", or when reasoning about how notemd notes relate. Explains that notemd's graph is derived (not an authorable file) and that argument maps are quote-anchored output that an agent may write only through notemd's set_argument_map tool (which validates every quote), never by fabricating or hand-editing the stored file.
---

# notemd Knowledge Graph & Argument Maps

notemd has two graph-like features. **Neither is a file you author by hand.** This skill exists so an agent understands what they are, how to influence them, and — critically — what not to fabricate. There is no `.canvas` / JSON Canvas equivalent in notemd; do not create one.

## The knowledge graph is derived, not authored

notemd's graph (shown in the **Knowledge Map**) is generated from the notes themselves. You cannot write nodes or edges directly. Edges come from three sources:

| Edge kind | Comes from |
|---|---|
| `articleLink` | Wikilinks `[[Name]]` between notes (see `notemd-markdown`) |
| `hierarchy` | Folder-note layout — `Foo.md` beside a `Foo/` folder parents its contents |
| `citation` | Citation links from a note to an indexed source (`[label](notemd://cite/<sourceID>…)`, see `notemd-research`) |

**To shape the graph, change the notes:** add or remove `[[wikilinks]]` and citation links, and organize folder notes. That is the only lever. Do not attempt to emit graph data structures.

## Argument maps are quote-anchored output, written only through notemd

An **argument map** is generated per indexed source document (typically a PDF). Either notemd's on-device extraction model builds it, or an agent connected to notemd's MCP server builds it and hands it to the app with **`set_argument_map`** (see the `notemd-research` skill; Premium, notemd open). Its structure (`ArgumentMapDocument`):

- **Nodes** with kinds: `central_claim`, `sub_claim`, `evidence`, `assumption`, `counterargument`, `rebuttal`, `limitation`, `scope_condition`, `cited_warrant`.
- **Edges** with kinds: `supports`, `qualifies`, `rebuts`, `assumes`, `evidences`.
- Each node carries a **verbatim quote anchor** tied to a specific page of the indexed source, plus a **confidence tier** (`exact` / `supported` / `weak`) that notemd's quote validator assigns — never a self-reported score.

**How notemd validates an agent-written map:** every node's quote is checked against the source's extracted full text and indexed chunks. A verbatim match grades `exact`, a match only after lowercasing/collapsing whitespace grades `supported`, an unmatched quote grades `weak`; a node with no quote (or one under 8 characters) is dropped, and edges to dropped or unknown parents are skipped. An unknown `kind`, empty `label` or duplicate `key` rejects the whole write. A map with user-edited nodes is only overwritten with `replace: true`, and the write is refused while the on-device model is generating the same map.

**Rules:**
- You **may** build an argument map by reading the source (`get_source_text`) and calling `set_argument_map` with real, verbatim quotes and pages. You can't set the tier; the app does.
- **Never fabricate or hand-edit the stored map file** (or any argument-map JSON on disk), and never invent quotes. The map's trustworthiness rests entirely on the anchors being real text from the source; bypassing the validator makes the confidence signal a lie.
- Don't overwrite the user's edits (`replace: true`) unless they asked.

## How you CAN help

- **Interpret / summarize** an existing argument map (`get_argument_map`) or graph the user shows you.
- **Build an argument map** for a source via `set_argument_map`, with verbatim quotes, when the user asks (or when one is queued in `list_agent_requests`).
- **Improve connectivity** by adding accurate `[[wikilinks]]` between related notes and organizing folder notes — this enriches the derived graph legitimately.
- **Explain** why a note is isolated in the graph (usually: nothing links to it and it links to nothing).
