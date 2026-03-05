---
name: button-system
version: 1.0.0
title: Button System
description: Build a complete multi-variant button component system with primary, secondary, danger, and ghost styles across small, medium, and large sizes.
author: ufira-ai
tags: [components, buttons, ui]
vibma_min: "0.1.0"
tools: [create_frame, create_text, styles, components]
---

# Button System

You are building a complete button component system in Figma. Follow every step in order. Do not skip ahead—each step produces IDs needed by the next.

## Step 1: Create Paint Styles for Backgrounds

Call `styles` once per style. Use `method: "create"` and `type: "paint"`.

```
styles({ method: "create", type: "paint", name: "Color/Primary/Default",   color: "#2563EB" })
styles({ method: "create", type: "paint", name: "Color/Primary/Hover",     color: "#1D4ED8" })
styles({ method: "create", type: "paint", name: "Color/Secondary/Default", color: "#F1F5F9" })
styles({ method: "create", type: "paint", name: "Color/Secondary/Hover",   color: "#E2E8F0" })
styles({ method: "create", type: "paint", name: "Color/Danger/Default",    color: "#DC2626" })
styles({ method: "create", type: "paint", name: "Color/Danger/Hover",      color: "#B91C1C" })
styles({ method: "create", type: "paint", name: "Color/Ghost/Default",     color: "#FFFFFF", opacity: 0 })
```

## Step 2: Create Paint Styles for Text

```
styles({ method: "create", type: "paint", name: "Color/Text/OnPrimary",   color: "#FFFFFF" })
styles({ method: "create", type: "paint", name: "Color/Text/OnSecondary", color: "#1E293B" })
styles({ method: "create", type: "paint", name: "Color/Text/OnDanger",    color: "#FFFFFF" })
styles({ method: "create", type: "paint", name: "Color/Text/OnGhost",     color: "#2563EB" })
```

Record every returned `styleId`—you will reference them when patching variant fills.

## Step 3: Build the 12 Variant Frames (4 variants × 3 sizes)

For each combination below, call `create_frame` with the exact properties listed. Name the frame using the `Variant=X, Size=Y` convention so Figma will auto-parse variant properties when the set is created.

### Size dimensions

| Size   | height | paddingLeft | paddingRight | paddingTop | paddingBottom | fontSize |
|--------|--------|-------------|--------------|------------|---------------|----------|
| Small  | 32     | 12          | 12           | 6          | 6             | 12       |
| Medium | 40     | 16          | 16           | 10         | 10            | 14       |
| Large  | 48     | 24          | 24           | 14         | 14            | 16       |

### Variant fill colors (use the styleId from Step 1 via `fillStyleId`)

| Variant   | fillStyleId key         | textStyleId key          |
|-----------|-------------------------|--------------------------|
| Primary   | Color/Primary/Default   | Color/Text/OnPrimary     |
| Secondary | Color/Secondary/Default | Color/Text/OnSecondary   |
| Danger    | Color/Danger/Default    | Color/Text/OnDanger      |
| Ghost     | Color/Ghost/Default     | Color/Text/OnGhost       |

Call `create_frame` like this (substitute actual values per combination):

```
create_frame({
  name: "Variant=Primary, Size=Medium",
  layoutMode: "HORIZONTAL",
  primaryAxisAlignItems: "CENTER",
  counterAxisAlignItems: "CENTER",
  primaryAxisSizingMode: "AUTO",
  counterAxisSizingMode: "FIXED",
  height: 40,
  paddingLeft: 16, paddingRight: 16, paddingTop: 10, paddingBottom: 10,
  cornerRadius: 6,
  fillStyleId: "<Color/Primary/Default styleId>"
})
```

Record the returned `nodeId` for each frame.

## Step 4: Add Label Text to Each Frame

For each of the 12 frames, call `create_text` with the frame's `nodeId` as `parentId`:

```
create_text({
  parentId: "<frame-nodeId>",
  characters: "Button",
  fontSize: <size-specific fontSize>,
  fontWeight: 500,
  textStyleId: "<variant-specific text styleId>"
})
```

Record the returned text `nodeId` for each frame—you need it for component properties.

## Step 5: Convert Each Frame to a Component

For each of the 12 frames:

```
components({
  method: "create",
  type: "from_node",
  nodeId: "<frame-nodeId>"
})
```

Record the returned component `nodeId`.

Then expose the label text as a component property:

```
components({
  method: "create",
  type: "component_property",
  nodeId: "<component-nodeId>",
  propertyName: "Label",
  propertyType: "TEXT",
  defaultValue: "Button",
  boundNodeId: "<text-nodeId>"
})
```

## Step 6: Combine Into a Variant Set

Collect all 12 component nodeIds. Call:

```
components({
  method: "create",
  type: "variant_set",
  componentIds: ["<id1>", "<id2>", ..., "<id12>"]
})
```

Figma infers `Variant` and `Size` properties automatically from the frame names set in Step 3. Record the returned `componentSetId`.

## Step 7: Lint the Variant Set

```
lint_node({
  nodeId: "<componentSetId>",
  rules: ["hardcoded-color", "no-text-style", "no-autolayout", "default-name"]
})
```

If `hardcoded-color` fires, the frame fill was not linked to a style—go back and verify `fillStyleId` was passed correctly. If `no-text-style` fires, verify `textStyleId` was passed to `create_text`. Fix all issues before reporting completion.

## Completion

Report to the user:
- The `componentSetId` of the final variant set
- The 4 variant names and 3 size names confirmed
- Any lint issues found and how they were resolved
