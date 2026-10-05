# notemd-knowledge-graph

An [Agent Skill](https://agentskills.io/specification) that teaches an agent how [notemd](https://note.md)'s graph features actually work — so it helps with them correctly and never fabricates data it shouldn't.

notemd has two graph-like features, and **neither is a file you author by hand**. Coming from Obsidian, an agent might reach for a `.canvas` file or try to emit graph nodes/edges — both wrong. This skill is a reference that sets expectations straight.

## What it covers

- **The knowledge graph is derived, not authored.** It's what the Knowledge Map shows, generated from the notes themselves — edges come from wikilinks (`articleLink`), folder-note layout (`hierarchy`), and citation links (`citation`). The only way to shape it is to change the notes; there is no graph file to edit and no JSON Canvas equivalent.
- **Argument maps are quote-anchored output.** notemd's on-device model builds them over an indexed source (typically a PDF), or an agent builds one and submits it through the MCP tool `set_argument_map`, where notemd re-checks every quote and assigns the **confidence tier** itself (Exact / Supported / Weak; nodes without a quote are dropped). The skill's core rule: an agent may write a map **only through `set_argument_map`** — never fabricate quotes or hand-edit the stored map file, because its trustworthiness depends entirely on those anchors being real.
- **How an agent *can* help** — interpreting an existing graph or map, building a map via `set_argument_map`, improving connectivity with accurate wikilinks, or explaining why a note is isolated.

## When it triggers

When the user asks about notemd's Knowledge Map (graph view), knowledge graph, backlinks, connections between notes, or "argument maps" — or when reasoning about how notemd notes relate.

## Related

Pairs with **notemd-markdown**, which covers the wikilink and folder-note syntax that *builds* the graph described here, and **notemd-research**, which covers the MCP tools (`get_argument_map`, `set_argument_map`) and setup.

## Installation

This skill follows the Agent Skills specification and works with Claude Code, Codex, and OpenCode.

**Manual (any agent):** copy this `notemd-knowledge-graph/` folder into your agent's skills directory:

- Claude Code — a `.claude/skills/` folder in your project (or vault) root
- Codex — `~/.codex/skills/`
- OpenCode — `~/.opencode/skills/`

The agent auto-discovers the `SKILL.md` inside. Restart the agent if it was already running.

**From a git repo:**

```
npx skills add <your-repo-url>
```

## Files

- `SKILL.md` — the skill itself (the detailed reference an agent loads)
- `README.md` — this human-facing overview

## License

MIT. Authored for notemd, in the style of [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills), © Steph Ango (@kepano) for the original collection.
