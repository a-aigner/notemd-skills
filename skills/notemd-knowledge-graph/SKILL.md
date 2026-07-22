---
name: notemd-knowledge-graph
description: Use when the user asks about notemd's graph view, knowledge graph, backlinks, connections between notes, or "argument maps", or when reasoning about how notemd notes relate. Explains that notemd's graph is derived (not an authorable file) and that argument maps are read-only AI output that must not be fabricated.
---

# notemd Knowledge Graph & Argument Maps (read-only reference)

notemd has two graph-like features. **Neither is a file you author.** This skill exists so an agent understands what they are, how to influence them, and — critically — what not to fabricate. There is no `.canvas` / JSON Canvas equivalent in notemd; do not create one.

## The knowledge graph is derived, not authored

notemd's graph view is generated from the notes themselves. You cannot write nodes or edges directly. Edges come from three sources:

| Edge kind | Comes from |
|---|---|
| `articleLink` | Wikilinks `[[Name]]` between notes (see `notemd-markdown`) |
| `hierarchy` | Folder-note layout — `Foo.md` beside a `Foo/` folder parents its contents |
| `citation` | Citation links to indexed sources (`notemd://cite/…`, created in-app) |

**To shape the graph, change the notes:** add or remove `[[wikilinks]]`, and organize folder notes. That is the only lever. Do not attempt to emit graph data structures.

## Argument maps are read-only AI output

An **argument map** is not authored by hand or by an agent. notemd generates it in-app by running an extraction model over an indexed source document (typically a PDF). Its structure (`ArgumentMapDocument`):

- **Nodes** with kinds: `central_claim`, `sub_claim`, `evidence`, `assumption`, `counterargument`, `rebuttal`, `limitation`, `scope_condition`, `cited_warrant`.
- **Edges** with kinds: `supports`, `qualifies`, `rebuts`, `assumes`, `evidences`.
- Each node carries a **verbatim quote anchor** tied to a specific page of the indexed source, plus a **confidence tier** (`exact` / `supported` / `weak` / `none`) derived from a post-hoc quote validator — not a self-reported score.

**Never fabricate or hand-edit an argument map.** Its trustworthiness rests entirely on those anchors being real quotes from indexed chunks; a hand-written map produces a confidence signal that is a lie. If the user asks to "make an argument map," direct them to notemd's in-app extraction over the source document.

## How you CAN help

- **Interpret / summarize** an existing argument map or graph the user shows you.
- **Improve connectivity** by adding accurate `[[wikilinks]]` between related notes and organizing folder notes — this enriches the derived graph legitimately.
- **Explain** why a note is isolated in the graph (usually: nothing links to it and it links to nothing).
