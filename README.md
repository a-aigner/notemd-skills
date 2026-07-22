Agent Skills for use with **notemd** (note.md).

These skills follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code, Codex, and Open Code.

They are adapted from [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) (MIT). notemd's on-disk vault is Obsidian-compatible in many respects (wikilinks, `> [!type]` callouts, `#tags`, KaTeX, folder-note hierarchy), but it renders only a subset of Obsidian syntax and has no Bases, JSON Canvas, or CLI. These skills document what notemd actually supports.

## Skills

| Skill | Description |
|-------|-------------|
| [notemd-markdown](skills/notemd-markdown) | Create and edit notemd Markdown (`.md`) — wikilinks, callouts, tags, math, frontmatter — covering only what notemd renders |
| [notemd-knowledge-graph](skills/notemd-knowledge-graph) | Read-only reference: notemd's graph is derived from links + folders; argument maps are AI output and must not be fabricated |
| [defuddle](skills/defuddle) | Extract clean markdown from web pages using Defuddle, removing clutter to save tokens (app-agnostic; unchanged from upstream) |

## Relationship to the Obsidian skills

Adapted from the 5 upstream Obsidian skills:

- **obsidian-markdown → notemd-markdown** — rewritten for notemd's supported subset (5 callout types; no embeds/mermaid/`%%`/`==`; notemd frontmatter schema).
- **defuddle → kept as-is** — app-agnostic web extraction.
- **json-canvas → replaced** by `notemd-knowledge-graph` — notemd has no `.canvas`; its graph is derived, not authored.
- **obsidian-bases → dropped** — notemd has no `.base` files.
- **obsidian-cli → dropped** — notemd has no CLI.

## License

MIT — see [LICENSE](LICENSE). Portions © Steph Ango (@kepano), adapted for notemd.
