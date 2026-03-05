---
name: icon-grid
version: 1.0.0
title: Icon Grid
description: Build a WRAP auto-layout icon grid with 24×24 icon frames, a base icon component, and instanced icons arranged in a responsive wrap layout.
author: ufira-ai
tags: [icons, layout, components]
vibma_min: "0.1.0"
tools: [create_frame, create_shape, components, patch_nodes]
---

# Icon Grid

You are building an icon grid system in Figma: a responsive WRAP container, a reusable 24×24 base icon component, and a set of instances arranged in a grid. Follow each step in order.

## Step 1: Create the Grid Container Frame

The container uses `layoutWrap: "WRAP"` so icons automatically reflow when the frame is resized.

```
create_frame({
  name: "Icon Grid",
  layoutMode: "HORIZONTAL",
  layoutWrap: "WRAP",
  primaryAxisAlignItems: "MIN",
  counterAxisAlignItems: "MIN",
  primaryAxisSizingMode: "FIXED",
  counterAxisSizingMode: "AUTO",
  width: 480,
  paddingLeft: 16, paddingRight: 16, paddingTop: 16, paddingBottom: 16,
  itemSpacing: 8,
  counterAxisSpacing: 8,
  fills: [{ type: "SOLID", color: "#F8FAFC" }],
  cornerRadius: 8
})
```

Record the returned `gridFrameId`.

## Step 2: Create the Base Icon Frame

Create a single 24×24 icon frame. This will become the main component that all instances are derived from.

```
create_frame({
  name: "Icon/Base",
  layoutMode: "HORIZONTAL",
  primaryAxisAlignItems: "CENTER",
  counterAxisAlignItems: "CENTER",
  primaryAxisSizingMode: "FIXED",
  counterAxisSizingMode: "FIXED",
  width: 24,
  height: 24,
  fills: [],
  clipsContent: true
})
```

Do NOT set a `parentId` here—create this frame at the top level so it can be a standalone component. Record the returned `iconFrameId`.

## Step 3: Add a Placeholder Shape Inside the Icon Frame

Add a centered vector placeholder to make the component visually non-empty:

```
create_shape({
  parentId: "<iconFrameId>",
  type: "RECTANGLE",
  name: "Icon Shape",
  width: 16,
  height: 16,
  cornerRadius: 2,
  fills: [{ type: "SOLID", color: "#94A3B8" }]
})
```

Record the returned `shapeNodeId`.

## Step 4: Convert the Icon Frame to a Component

```
components({
  method: "create",
  type: "from_node",
  nodeId: "<iconFrameId>"
})
```

Record the returned `iconComponentId`. This is the main component for all icon instances in the grid.

## Step 5: Expose the Icon Color as a Component Property

To allow per-instance color overrides, patch the shape to use a component property. First expose a fill variable on the shape:

```
patch_nodes({
  nodeId: "<shapeNodeId>",
  fills: [{ type: "SOLID", color: "#94A3B8" }]
})
```

Then expose a component property for the icon color:

```
components({
  method: "create",
  type: "component_property",
  nodeId: "<iconComponentId>",
  propertyName: "Color",
  propertyType: "TEXT",
  defaultValue: "#94A3B8",
  boundNodeId: "<shapeNodeId>"
})
```

## Step 6: Populate the Grid with Instances

Create icon instances inside the grid container. Repeat the following call for as many icons as needed (typically 12–48 for a seed grid). The grid container's WRAP layout will arrange them automatically.

```
instances({
  method: "create",
  componentId: "<iconComponentId>",
  parentId: "<gridFrameId>"
})
```

Call this N times (where N is the number of icons requested by the user, default 24). Do not set `x` or `y`—the WRAP auto-layout positions all instances automatically based on `itemSpacing` and `counterAxisSpacing` set in Step 1.

Record each instance's `nodeId` in an array `instanceIds`.

## Step 7: Name Each Instance

After creation, give each instance a descriptive name so they are not flagged by the `default-name` lint rule. If the user has provided icon names, use them; otherwise use sequential names:

```
patch_nodes({ nodeId: "<instanceId-0>", name: "Icon/Arrow Right" })
patch_nodes({ nodeId: "<instanceId-1>", name: "Icon/Arrow Left" })
patch_nodes({ nodeId: "<instanceId-2>", name: "Icon/Arrow Up" })
...
```

If the user has not specified names, use the pattern `Icon/<N>` (e.g., `Icon/1`, `Icon/2`, …).

## Step 8: Adjust Grid Width if Needed

The grid container is fixed at 480px. If the user wants a different number of columns, calculate the correct width:

```
width = (columns × 24) + ((columns - 1) × itemSpacing) + paddingLeft + paddingRight
      = (columns × 24) + ((columns - 1) × 8) + 32
```

Then resize the container:

```
patch_nodes({
  nodeId: "<gridFrameId>",
  width: <calculated-width>
})
```

The WRAP layout will reflow instances to fit the new width automatically.

## Step 9: Lint the Grid

```
lint_node({
  nodeId: "<gridFrameId>",
  rules: ["hardcoded-color", "no-text-style", "no-autolayout", "default-name"]
})
```

Expected results:
- `no-autolayout`: should not fire—the grid and icon frames both use `layoutMode: "HORIZONTAL"`
- `default-name`: should not fire if Step 7 was completed
- `hardcoded-color`: may fire on the placeholder shapes—this is acceptable for placeholder content; note it to the user

## Layout Properties Summary

| Property                  | Grid Container | Icon Frame |
|---------------------------|---------------|------------|
| `layoutMode`              | HORIZONTAL    | HORIZONTAL |
| `layoutWrap`              | WRAP          | NO_WRAP    |
| `primaryAxisAlignItems`   | MIN           | CENTER     |
| `counterAxisAlignItems`   | MIN           | CENTER     |
| `width`                   | 480 (fixed)   | 24 (fixed) |
| `height`                  | AUTO          | 24 (fixed) |
| `itemSpacing`             | 8             | 0          |
| `counterAxisSpacing`      | 8             | —          |

## Completion

Report to the user:
- The `gridFrameId` and icon `componentId`
- Number of instances created
- Grid dimensions and column count
- Any lint issues found and resolution
