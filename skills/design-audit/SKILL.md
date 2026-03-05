---
name: design-audit
version: 1.0.0
title: Design Audit
description: Run a systematic design audit using lint_node across an entire selection or page, group issues by rule, and produce a prioritized report.
author: ufira-ai
tags: [audit, lint, quality]
vibma_min: "0.1.0"
tools: [lint_node, get_node_info, get_selection, search_nodes]
---

# Design Audit

You are running a systematic design audit on a Figma file. This skill uses `lint_node`, `get_selection`, `get_node_info`, and `search_nodes` to surface issues, group them by rule type, and produce a prioritized remediation report.

## Lint Rules Reference

Vibma supports four lint rules. Know what each catches before you begin:

| Rule              | What it flags                                                                 | Severity |
|-------------------|-------------------------------------------------------------------------------|----------|
| `hardcoded-color` | Fills or strokes using raw hex/rgba values instead of a style or variable     | High     |
| `no-text-style`   | Text nodes with manually set font properties not linked to a text style       | High     |
| `no-autolayout`   | Frames using manual positioning (no `layoutMode`) where auto-layout is viable | Medium   |
| `default-name`    | Nodes still named with Figma's generated defaults (Frame 1, Rectangle 3, …)  | Low      |

## Step 1: Determine Audit Scope

First ask the user: "Audit the current selection, a specific frame by name, or the entire page?"

**Option A — Current selection:**
```
get_selection()
```
Extract the `nodeId` from the first selected node. If nothing is selected, fall back to Option C.

**Option B — Named frame:**
```
search_nodes({
  query: "<frame-name>",
  nodeType: "FRAME"
})
```
Use the first matching `nodeId`.

**Option C — Entire page (scan for top-level frames):**
```
search_nodes({
  nodeType: "FRAME",
  depth: 1
})
```
This returns all top-level frames. You will lint each one individually and aggregate results.

Record the target `nodeId`(s). For a page-wide audit you may have many frames—lint them all.

## Step 2: Gather Node Metadata

Before linting, get context on what you're auditing:

```
get_node_info({ nodeId: "<targetNodeId>" })
```

Note the node's `name`, `type`, `childCount`, and whether it is a component or component set. This helps contextualize the lint results.

## Step 3: Run lint_node on Each Target

For each target node:

```
lint_node({
  nodeId: "<targetNodeId>",
  rules: ["hardcoded-color", "no-text-style", "no-autolayout", "default-name"]
})
```

`lint_node` returns an array of issues. Each issue has:
- `rule`: the rule name
- `nodeId`: the specific node with the issue
- `nodeName`: human-readable name
- `message`: description of the problem
- `severity`: high / medium / low

Collect all issues across all linted nodes into a flat array.

## Step 4: Group Issues by Rule

Organize issues into four buckets:

```
hardcodedColors  = issues.filter(i => i.rule === "hardcoded-color")
noTextStyles     = issues.filter(i => i.rule === "no-text-style")
noAutolayout     = issues.filter(i => i.rule === "no-autolayout")
defaultNames     = issues.filter(i => i.rule === "default-name")
```

## Step 5: Identify Repeated Offenders

Within `hardcodedColors` and `noTextStyles`, look for nodes that appear more than once or that share the same parent component. These indicate a systemic problem (e.g., an entire component family using raw colors) rather than a one-off mistake. Flag these separately as "systemic issues."

## Step 6: Produce the Structured Report

Output a report in this exact format:

---

### Design Audit Report

**Scope:** `<node name>` (`<nodeId>`)
**Total issues found:** `<N>`

#### High Priority

**Hardcoded Colors** — `<count>` issues
| Node Name | Node ID | Message |
|-----------|---------|---------|
| ...       | ...     | ...     |

**Missing Text Styles** — `<count>` issues
| Node Name | Node ID | Message |
|-----------|---------|---------|
| ...       | ...     | ...     |

#### Medium Priority

**Missing Auto-Layout** — `<count>` issues
| Node Name | Node ID | Message |
|-----------|---------|---------|
| ...       | ...     | ...     |

#### Low Priority

**Default Names** — `<count>` issues
| Node Name | Node ID | Message |
|-----------|---------|---------|
| ...       | ...     | ...     |

#### Systemic Issues
List any components or families where the same rule fires on more than 3 sibling nodes—these likely need a batch fix at the component level, not node-by-node.

#### Recommended Fix Order
1. Fix `hardcoded-color` in components first—changes propagate to all instances automatically.
2. Fix `no-text-style`—apply or create the correct text style via `styles` and link it.
3. Fix `no-autolayout`—convert manual-positioned frames using `patch_nodes({ nodeId, layoutMode: "VERTICAL" })`.
4. Fix `default-name` last—low impact, use `patch_nodes({ nodeId, name: "<meaningful name>" })`.

---

## Step 7: Offer to Fix Issues

After presenting the report, ask: "Should I fix any of these automatically?"

If the user confirms:
- **Hardcoded color**: Use `patch_nodes` to set `fillStyleId` or `strokeStyleId` to the nearest matching style. If no style exists, create one with `styles({ method: "create", type: "paint", ... })` first.
- **Missing text style**: Use `patch_nodes({ nodeId, textStyleId: "<styleId>" })`. If no style matches, create one with `styles({ method: "create", type: "text", ... })`.
- **Default name**: Use `patch_nodes({ nodeId, name: "<meaningful name>" })`. Derive the name from the node's content or parent context.
- **No auto-layout**: Use `patch_nodes({ nodeId, layoutMode: "VERTICAL", primaryAxisAlignItems: "MIN", counterAxisAlignItems: "MIN" })`. Only do this for frames where auto-layout is safe to add (i.e., children are stacked, not overlapping).

Re-run `lint_node` after fixes to confirm the issue count has dropped.

## Completion

Report final issue count before and after any fixes applied. If no fixes were requested, summarize next steps the user can take manually.
