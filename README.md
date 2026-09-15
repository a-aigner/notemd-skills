Agent Skills for use with **notemd** (note.md).

These skills follow the [Agent Skills specification](https://agentskills.io/specification) so they can be used by any skills-compatible agent, including Claude Code, Codex, and Open Code.

They are adapted from [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) (MIT). notemd's on-disk vault is Obsidian-compatible in many respects (wikilinks, `> [!type]` callouts, `#tags`, KaTeX, folder-note hierarchy), but it renders only a subset of Obsidian syntax and has no Bases or JSON Canvas. Since 1.5 it has a CLI and MCP server, covered by `notemd-research`. These skills document what notemd actually supports.

## Skills

| Skill | Description |
|-------|-------------|
| [notemd-markdown](skills/notemd-markdown) | Create and edit notemd Markdown (`.md`) — wikilinks, callouts, tags, math, frontmatter — covering only what notemd renders |
| [notemd-knowledge-graph](skills/notemd-knowledge-graph) | Read-only reference: notemd's graph is derived from links + folders; argument maps are AI output and must not be fabricated |
| [notemd-research](skills/notemd-research) | Drive note.md's MCP server / CLI: search notes and the real PDFs, read sources, cite page-level bookmarks, file and tag sources (note.md 1.5+, Premium) |

## License

MIT — see [LICENSE](LICENSE). Portions © Steph Ango (@kepano), adapted for notemd.
