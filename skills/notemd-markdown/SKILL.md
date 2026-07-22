---
name: notemd-markdown
description: Use when creating or editing .md files in a notemd vault (the macOS/iOS note app that stores Markdown on disk), or when the user mentions notemd notes, wikilinks, callouts, tags, math, or frontmatter in notemd. notemd reads Obsidian-style Markdown but renders only a subset — this skill covers exactly what works so notes don't silently render wrong.
---

# notemd Flavored Markdown

notemd stores each note as a `.md` file in a per-project vault. On disk it uses **Obsidian-compatible** syntax (wikilinks, `> [!type]` callouts, `#tags`, KaTeX), but its renderer supports only a **subset** of Obsidian. Writing an unsupported construct (mermaid, `![[embeds]]`, `%%comments%%`, `==highlight==`) is silent — the file saves, but the note renders the raw text. This skill covers only what notemd actually renders. Standard Markdown (headings, bold, italic, lists, quotes, code, tables) works as normal and is assumed knowledge.

## What notemd supports vs. Obsidian

| Works in notemd | Does NOT render in notemd |
|---|---|
| Wikilinks `[[Name]]`, `[[Folder/Name]]`, `[[Name\|Alias]]` | Embeds `![[Note]]`, `![[image.png]]` |
| Callouts `> [!type]` — 5 types only (see below) | Obsidian callout types outside those 5 |
| `#tags`, frontmatter `tags` | `%%comments%%` |
| KaTeX math `$…$`, `$$…$$` | `==highlight==` |
| Footnotes, task lists, tables, `<details>` toggles | Mermaid diagrams |

## Internal links (wikilinks)

notemd resolves wikilinks by Obsidian's **closest-note / shortest-path** rule and tracks renames automatically (each note carries a durable internal `id`).

```markdown
[[Note Name]]                 Link to the closest note named "Note Name"
[[Folder/Note Name]]          Disambiguate when two notes share a name
[[Note Name|Display Text]]    Custom link text
```

- Use `[[Folder/Name]]` **only** to disambiguate — if a name is unique, bare `[[Name]]` is preferred and is what notemd rewrites links to on save.
- Do **not** hand-write `[label](notemd://article/…)` links. That `notemd://` form is notemd's internal/rendered representation; author the `[[ ]]` form and notemd converts it.
- Linking notes is also what builds notemd's graph view — see the `notemd-knowledge-graph` skill.

## Frontmatter (properties)

notemd manages a fixed frontmatter schema:

```yaml
---
title: My Note
tags:
  - project
  - active
created_at: 2026-07-06T10:00:00Z
updated_at: 2026-07-06T10:00:00Z
---
```

- **`id`** — a UUID that is notemd's durable identity. **notemd assigns it; never write or copy one by hand.** Reusing another note's `id` creates a duplicate-identity collision that can crash merge and the graph. For a brand-new file, omit `id` — notemd assigns one on import.
- **`title`** — optional; if omitted, the filename is the title. The body should therefore **not** repeat the title as an `#` H1 — start body headings at `##`.
- **`tags`** — a YAML list. Also usable inline in the body as `#tag`.
- **`created_at` / `updated_at`** — ISO 8601; notemd maintains these.
- Extra keys are preserved but ignored (as in Obsidian).

## Hierarchy (folder notes)

notemd derives a note's parent from **folder layout**, not a frontmatter field. A note `Foo.md` placed beside a `Foo/` folder becomes the parent of every note inside `Foo/`. Plain folders with no matching `.md` are transparent pass-throughs.

## Callouts

Blockquote form, first line `[!type]` with an optional title:

```markdown
> [!warning] Optional Title
> Body of the callout.
```

Supported types — **only these five**; any other type falls back to `note`:

`note`, `tip`, `important`, `warning`, `caution`

(These are the GitHub-alert set. Obsidian types like `danger`, `bug`, `faq`, `success`, `question`, `quote`, `abstract` are **not** distinct in notemd — they all render as `note`.)

## Tags

```markdown
#tag                    Inline tag
#nested/tag             Nested tag
```

Tags are searchable (`#tag` in global search) and may also be declared in frontmatter `tags`.

## Math (KaTeX)

```markdown
Inline: $e^{i\pi} + 1 = 0$

$$
\frac{a}{b} = c
$$
```

## Footnotes, task lists, tables, toggles

```markdown
Text with a footnote.[^1]

[^1]: Footnote body.

- [x] Done
- [ ] Not done

<details>
<summary>Click to expand</summary>

Hidden content.

</details>
```

Standard GFM tables render normally.

## Complete example

```markdown
---
title: Project Alpha
tags:
  - project
  - active
---

## Overview

This project builds on [[Improve Workflow]] and the [[Research/Sorting|sorting notes]].

> [!important] Key Deadline
> First milestone is due January 30th.

## Tasks

- [x] Initial planning
- [ ] Development

The algorithm runs in $O(n \log n)$. See [[Algorithm Notes]] for the proof.[^1]

[^1]: Derived from the master theorem.
```

## Common mistakes

- **`==highlight==`, `%%comment%%`, mermaid blocks, `![[embeds]]`** — none render; they appear as literal text. Don't use them.
- **Writing a raw `notemd://article/…` link** — author `[[Name]]` instead.
- **Hand-setting `id` in frontmatter** — never; it risks a duplicate-identity crash. Omit it and let notemd assign.
- **An `#` H1 that repeats the title** — the title comes from frontmatter/filename; start the body at `##`.
- **Unsupported callout type** (`[!danger]`, `[!info]`, …) — silently degrades to `note`. Pick one of the five.
