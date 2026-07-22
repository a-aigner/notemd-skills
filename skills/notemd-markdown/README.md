# notemd-markdown

An [Agent Skill](https://agentskills.io/specification) for creating and editing Markdown notes in a [notemd](https://note.md) vault — covering **only the syntax notemd actually renders**, so an agent never writes constructs that silently fail.

notemd stores each note as a `.md` file. On disk it uses **Obsidian-compatible** syntax (wikilinks, `> [!type]` callouts, `#tags`, KaTeX), but its renderer supports only a **subset** of Obsidian. Unsupported constructs (`![[embeds]]`, Mermaid, `%%comments%%`, `==highlight==`) don't error — the file saves and the note just renders the raw text. This skill pins the boundary so notes render correctly the first time.

## What it covers

- **Wikilinks** — `[[Name]]`, `[[Folder/Name]]`, `[[Name|Alias]]` with notemd's closest-note / shortest-path resolution, and why you never hand-write the internal `notemd://article/…` form.
- **Frontmatter** — notemd's managed schema (`id`, `title`, `created_at`, `updated_at`, `tags`) and the **critical rule that `id` is app-managed** — hand-writing one risks a duplicate-identity crash.
- **Folder-note hierarchy** — how a note's parent is derived from folder layout (`Foo.md` beside a `Foo/` folder), not a frontmatter field.
- **Callouts** — the five supported types (`note`, `tip`, `important`, `warning`, `caution`) and how every other Obsidian type silently degrades to `note`.
- **Tags, KaTeX math, footnotes, task lists, tables, `<details>` toggles** — the supported extras.
- **What does NOT render** — an explicit "don't use these" list.

## When it triggers

When creating or editing `.md` files in a notemd vault, or when the user mentions notemd notes, wikilinks, callouts, tags, math, or frontmatter in notemd.

## Installation

This skill follows the Agent Skills specification and works with Claude Code, Codex, and OpenCode.

**Manual (any agent):** copy this `notemd-markdown/` folder into your agent's skills directory:

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

MIT. Adapted for notemd from [kepano/obsidian-skills](https://github.com/kepano/obsidian-skills) (`obsidian-markdown`), © Steph Ango (@kepano).
