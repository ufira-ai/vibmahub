# VibmaHub

VibmaHub is the community skill and prompt registry for [Vibma](https://github.com/ufira-ai/vibma) — an MCP server that exposes Figma as tools to AI coding agents like Claude Code and Cursor. Skills are additional MCP prompts that extend what Claude knows how to do in Figma. They don't add new tool capabilities; instead they give Claude step-by-step, tool-specific workflows so it can tackle common Figma tasks end-to-end without improvising.

## Using Skills

### 1. Load skills into Vibma

Point Vibma at this registry (or individual skill files) in your MCP config. See the [Vibma docs](https://github.com/ufira-ai/vibma) for the exact CLI flags and config format. Once loaded, each skill registers as a named MCP prompt alongside Vibma's five built-in prompts.

### 2. Invoke a skill in your agent

Once skills are registered, ask Claude to use one by name — no special syntax required:

```
Build a button system using the button-system skill.
```

```
Run a design audit on my current selection.
```

```
Use the color-system skill to set up tokens for this file.
```

Claude receives the skill's full prompt body as a system-level instruction and immediately knows which Vibma tools to call, in what order, and with what parameters. You don't need to explain the workflow — that's what the skill is for.

### 3. How it works under the hood

When Vibma loads a skill, it parses the `SKILL.md` frontmatter to extract metadata (name, tools, version) and registers the Markdown body as an MCP prompt. The prompt body is written as a direct instruction to Claude — not documentation — so it reads like a system message telling Claude exactly what to do. Skills reference Vibma's exact tool names (`create_frame`, `styles`, `lint_node`, etc.) and parameter shapes, so Claude knows precisely which tool to call at each step without improvising.

## Browsing the Registry

- **Machine-readable index:** [`registry.json`](./registry.json) — auto-generated on every push to `main`
- **Human-readable skills:** [`skills/`](./skills/) — one directory per skill, each containing a `SKILL.md`

## Seed Skills

| Slug | Title | Description | Tags |
|------|-------|-------------|------|
| `button-system` | Button System | Build a complete multi-variant button component system with primary, secondary, danger, and ghost styles across small, medium, and large sizes. | `components` `buttons` `ui` |
| `card-component` | Card Component | Build a card component with an image placeholder, title, body text, and a CTA button instance using auto-layout and exposed text properties. | `components` `cards` `layout` |
| `color-system` | Color System | Build a semantic color system with primitive paint styles and semantic design tokens as Figma variables supporting light and dark modes. | `design-tokens` `colors` `variables` |
| `design-audit` | Design Audit | Run a systematic design audit using lint_node across an entire selection or page, group issues by rule, and produce a prioritized report. | `audit` `lint` `quality` |
| `icon-grid` | Icon Grid | Build a WRAP auto-layout icon grid with 24×24 icon frames, a base icon component, and instanced icons arranged in a responsive wrap layout. | `icons` `layout` `components` |

## Skill Format (Abbreviated)

Each skill is a `SKILL.md` file with a YAML frontmatter block followed by a Markdown prompt body:

```markdown
---
name: my-skill
version: 1.0.0
title: My Skill
description: One sentence.
author: your-github-handle
tags: [tag1, tag2]
vibma_min: "0.1.0"
tools: [create_frame, styles]
---

# My Skill

Full prompt body — written as a direct instruction to Claude. Reference exact
Vibma tool names (create_frame, styles, components, …) and their parameters.
Include step-by-step workflows with expected inputs and outputs.
```

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for the full field reference and quality bar.

## Contributing

See [`CONTRIBUTING.md`](./CONTRIBUTING.md) for how to add or improve a skill.
