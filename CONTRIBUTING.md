# Contributing to VibmaHub

VibmaHub is a community registry. Anyone can submit a skill. This guide covers everything you need to write a high-quality skill and get it merged.

## Before You Start

Read a few existing skills in [`skills/`](./skills/) to calibrate the quality bar. Notice that every skill:

- References **exact** Vibma tool names (`create_frame`, not "create a frame")
- Specifies **exact** parameter names and values (`layoutMode: "HORIZONTAL"`, not "enable horizontal layout")
- Provides a complete, end-to-end workflow a model can follow without improvising
- Handles error cases or follow-up actions (e.g., re-running lint after fixes)

A skill that gives generic Figma advice will be rejected.

## Creating a New Skill

### 1. Fork and clone the repo

```bash
git clone https://github.com/ufira-ai/vibmahub.git
cd vibmahub
git checkout -b skill/my-skill-name
```

### 2. Create the skill directory

```
skills/
└── my-skill-name/
    └── SKILL.md
```

The directory name becomes the skill's `slug`. Use kebab-case. Pick a name that is specific and searchable (e.g., `spacing-tokens`, `dark-mode-swap`, `responsive-grid`).

### 3. Write SKILL.md

Use the template below. Every field in the frontmatter is required unless marked optional.

```markdown
---
name: my-skill-name
version: 1.0.0
title: My Skill Title
description: One sentence describing what this skill helps Claude accomplish.
author: your-github-handle
tags: [tag1, tag2]
vibma_min: "0.1.0"
tools: [create_frame, styles]
---

# My Skill Title

<prompt body>
```

### 4. Open a pull request

```bash
git add skills/my-skill-name/
git commit -m "feat: add my-skill-name skill"
git push origin skill/my-skill-name
```

Open a PR against `main`. The `registry.json` is regenerated automatically by the GitHub Action when your PR is merged — do not edit it manually.

---

## Frontmatter Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | string | yes | Must match the directory name exactly (kebab-case slug) |
| `version` | string | yes | Semver string, e.g. `"1.0.0"` |
| `title` | string | yes | Human-readable title shown in UIs, max ~50 chars |
| `description` | string | yes | One sentence (no period at end), max ~160 chars |
| `author` | string | yes | GitHub handle of the primary author |
| `tags` | string[] | yes | 2–5 lowercase kebab-case tags, e.g. `[components, layout]` |
| `vibma_min` | string | yes | Minimum Vibma version required, semver, e.g. `"0.1.0"` |
| `tools` | string[] | yes | Exact Vibma tool names used in the prompt body |

### Valid tool names

These are the Vibma tools skills may reference. Only list tools your skill actually uses.

```
create_frame      create_text       create_shape
patch_nodes       set_text_content  styles
variables         components        instances
lint_node         get_node_info     get_selection
export_node_as_image  fonts         search_nodes
```

### Valid tag examples

```
components  buttons  cards  forms  layout  icons  typography
colors  design-tokens  variables  audit  lint  quality
spacing  animation  dark-mode  responsive  ui  navigation
```

---

## The Quality Bar

A skill must pass all of the following before it will be merged:

**1. Specificity over generality**
Every instruction must name the exact tool and exact parameter. "Set the fill" is not acceptable. `patch_nodes({ nodeId, fills: [{ type: "SOLID", color: "#2563EB" }] })` is acceptable.

**2. End-to-end completeness**
The workflow must start from zero (or a clearly stated prerequisite) and end with a usable result. Claude should not need to improvise any step.

**3. ID propagation**
If a step produces a `nodeId`, the skill must explicitly tell Claude to record it and show how it is used in the next step. Never assume the model remembers IDs across steps without being told.

**4. Lint sign-off**
Workflows that produce nodes must end with a `lint_node` call covering at minimum `hardcoded-color` and `no-text-style`. The skill must instruct Claude to resolve any issues before reporting completion.

**5. No invented tools**
Only reference tools from the valid tool names list above. Do not invent parameter names that don't exist.

**6. No generic Figma advice**
Advice like "make sure your colors are accessible" or "use consistent spacing" is not a skill — it's documentation. Skills give Claude executable instructions, not philosophy.

---

## Versioning

Skills use [semver](https://semver.org/):

| Change type | Version bump |
|---|---|
| New step added, new parameter documented, workflow extended | `minor` (1.0.0 → 1.1.0) |
| Existing step corrected, parameter name fixed, tool name updated | `patch` (1.0.0 → 1.0.1) |
| Workflow restructured, scope changed significantly, breaking change to expected behavior | `major` (1.0.0 → 2.0.0) |

Always bump the version in the frontmatter when submitting changes to an existing skill.

---

## What Not to Submit

- Skills that duplicate an existing one with only minor wording changes — improve the existing skill instead
- Skills that only work for one specific Figma file or team library
- Skills that require npm packages, external APIs, or anything beyond Vibma's built-in tools
- Skills with placeholder content like "TODO" or "add your steps here"
- Skills with a prompt body under ~200 words — if the workflow can be described that briefly, it's probably too generic

---

## Review Process

All PRs are reviewed for the quality bar above. Expect feedback on:
- Parameter specificity
- Missing ID propagation
- Steps that require Claude to improvise
- Missing lint step

Maintainers may request changes before merging. Once merged, `registry.json` is regenerated automatically within minutes.
