# notemd-knowledge-graph

An [Agent Skill](https://agentskills.io/specification) that teaches an agent how [notemd](https://note.md)'s graph features actually work — so it helps with them correctly and never fabricates data it shouldn't.

notemd has two graph-like features, and **neither is a file you author**. Coming from Obsidian, an agent might reach for a `.canvas` file or try to emit graph nodes/edges — both wrong. This skill is a **read-only reference** that sets expectations straight.

## What it covers

- **The knowledge graph is derived, not authored.** It's generated from the notes themselves — edges come from wikilinks (`articleLink`), folder-note layout (`hierarchy`), and citation links (`citation`). The only way to shape it is to change the notes; there is no graph file to edit and no JSON Canvas equivalent.
- **Argument maps are read-only AI output.** notemd generates them in-app by running an extraction model over an indexed source (typically a PDF). Each node carries a **verbatim quote anchor** tied to a source page plus a validator-derived **confidence tier**. The skill's core rule: **never fabricate or hand-edit an argument map** — its trustworthiness depends entirely on those anchors being real.
- **How an agent *can* help** — interpreting an existing graph or map, improving connectivity with accurate wikilinks, or explaining why a note is isolated.

## When it triggers

When the user asks about notemd's graph view, knowledge graph, backlinks, connections between notes, or "argument maps" — or when reasoning about how notemd notes relate.

## Related

Pairs with **notemd-markdown**, which covers the wikilink and folder-note syntax that *builds* the graph described here.

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
