---
name: color-system
version: 1.0.0
title: Color System
description: Build a semantic color system with primitive paint styles and semantic design tokens as Figma variables supporting light and dark modes.
author: ufira-ai
tags: [design-tokens, colors, variables]
vibma_min: "0.1.0"
tools: [styles, variables]
---

# Color System

You are building a two-layer color system: primitive paint styles (raw values) and semantic variables (light/dark tokens). Follow this order exactly—variables reference style IDs produced in earlier steps.

## Layer 1: Primitive Paint Styles

Create all primitive colors as paint styles using `styles`. These are the raw palette—not applied directly to components, only referenced by semantic tokens.

### Blues (Primary scale)

```
styles({ method: "create", type: "paint", name: "Color/Blue/50",  color: "#EFF6FF" })
styles({ method: "create", type: "paint", name: "Color/Blue/100", color: "#DBEAFE" })
styles({ method: "create", type: "paint", name: "Color/Blue/500", color: "#3B82F6" })
styles({ method: "create", type: "paint", name: "Color/Blue/600", color: "#2563EB" })
styles({ method: "create", type: "paint", name: "Color/Blue/700", color: "#1D4ED8" })
styles({ method: "create", type: "paint", name: "Color/Blue/900", color: "#1E3A8A" })
```

### Neutrals (Slate scale)

```
styles({ method: "create", type: "paint", name: "Color/Slate/0",   color: "#FFFFFF" })
styles({ method: "create", type: "paint", name: "Color/Slate/50",  color: "#F8FAFC" })
styles({ method: "create", type: "paint", name: "Color/Slate/100", color: "#F1F5F9" })
styles({ method: "create", type: "paint", name: "Color/Slate/200", color: "#E2E8F0" })
styles({ method: "create", type: "paint", name: "Color/Slate/400", color: "#94A3B8" })
styles({ method: "create", type: "paint", name: "Color/Slate/600", color: "#475569" })
styles({ method: "create", type: "paint", name: "Color/Slate/700", color: "#334155" })
styles({ method: "create", type: "paint", name: "Color/Slate/900", color: "#0F172A" })
styles({ method: "create", type: "paint", name: "Color/Slate/950", color: "#020617" })
```

### Feedback colors

```
styles({ method: "create", type: "paint", name: "Color/Red/50",  color: "#FEF2F2" })
styles({ method: "create", type: "paint", name: "Color/Red/500", color: "#EF4444" })
styles({ method: "create", type: "paint", name: "Color/Red/600", color: "#DC2626" })
styles({ method: "create", type: "paint", name: "Color/Green/50",  color: "#F0FDF4" })
styles({ method: "create", type: "paint", name: "Color/Green/500", color: "#22C55E" })
styles({ method: "create", type: "paint", name: "Color/Green/600", color: "#16A34A" })
styles({ method: "create", type: "paint", name: "Color/Amber/50",  color: "#FFFBEB" })
styles({ method: "create", type: "paint", name: "Color/Amber/500", color: "#F59E0B" })
```

Record all returned `styleId` values in a map keyed by the style name.

## Layer 2: Create the Variable Collection with Modes

Call `variables` to create a collection named `"Color Tokens"` with two modes:

```
variables({
  method: "create",
  type: "collection",
  name: "Color Tokens",
  modes: ["Light", "Dark"]
})
```

Record the returned `collectionId` and the `modeId` for each mode (`lightModeId`, `darkModeId`).

## Layer 3: Create Semantic Variables

For each semantic token below, call `variables` with `method: "create"` and `type: "variable"`. Set `collectionId`, `variableType: "COLOR"`, and `modeValues` mapping each mode to a primitive hex.

### Primary tokens

| Variable name             | Light value   | Dark value    |
|---------------------------|---------------|---------------|
| `color/primary/default`   | `#2563EB`     | `#3B82F6`     |
| `color/primary/hover`     | `#1D4ED8`     | `#2563EB`     |
| `color/primary/subtle`    | `#EFF6FF`     | `#1E3A8A`     |
| `color/primary/on`        | `#FFFFFF`     | `#FFFFFF`     |

```
variables({
  method: "create",
  type: "variable",
  collectionId: "<collectionId>",
  name: "color/primary/default",
  variableType: "COLOR",
  modeValues: {
    "<lightModeId>": "#2563EB",
    "<darkModeId>":  "#3B82F6"
  }
})
```

Repeat for all primary tokens.

### Surface tokens

| Variable name             | Light value   | Dark value    |
|---------------------------|---------------|---------------|
| `color/surface/page`      | `#FFFFFF`     | `#0F172A`     |
| `color/surface/card`      | `#F8FAFC`     | `#1E293B`     |
| `color/surface/overlay`   | `#F1F5F9`     | `#334155`     |

### Text tokens

| Variable name             | Light value   | Dark value    |
|---------------------------|---------------|---------------|
| `color/text/primary`      | `#0F172A`     | `#F8FAFC`     |
| `color/text/secondary`    | `#475569`     | `#94A3B8`     |
| `color/text/disabled`     | `#94A3B8`     | `#475569`     |
| `color/text/inverse`      | `#FFFFFF`     | `#0F172A`     |

### Border tokens

| Variable name             | Light value   | Dark value    |
|---------------------------|---------------|---------------|
| `color/border/default`    | `#E2E8F0`     | `#334155`     |
| `color/border/strong`     | `#94A3B8`     | `#475569`     |

### Feedback tokens

| Variable name              | Light value   | Dark value    |
|----------------------------|---------------|---------------|
| `color/feedback/error`     | `#DC2626`     | `#EF4444`     |
| `color/feedback/error-bg`  | `#FEF2F2`     | `#450A0A`     |
| `color/feedback/success`   | `#16A34A`     | `#22C55E`     |
| `color/feedback/success-bg`| `#F0FDF4`     | `#052E16`     |
| `color/feedback/warning`   | `#F59E0B`     | `#F59E0B`     |
| `color/feedback/warning-bg`| `#FFFBEB`     | `#451A03`     |

## Layer 4: Bind Variables to Paint Styles (Optional but Recommended)

For each primitive paint style that has a matching semantic variable, update the style to reference the variable so Figma surfaces the token in the inspector.

```
styles({
  method: "update",
  styleId: "<Color/Blue/600 styleId>",
  boundVariableId: "<color/primary/default variableId>"
})
```

Only bind styles to their closest semantic match. Do not bind every primitive—only those directly used in components.

## Completion

Report to the user:
- Collection name, mode names, and total variable count
- A summary table of all semantic token names grouped by category (primary, surface, text, border, feedback)
- Any variables that could not be created and why
