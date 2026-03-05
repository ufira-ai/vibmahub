---
name: card-component
version: 1.0.0
title: Card Component
description: Build a card component with an image placeholder, title, body text, and a CTA button instance using auto-layout and exposed text properties.
author: ufira-ai
tags: [components, cards, layout]
vibma_min: "0.1.0"
tools: [create_frame, create_text, components, styles, variables, instances]
---

# Card Component

You are building a reusable card component in Figma. The card contains an image placeholder, a title, body text, and a CTA button instance. Follow each step in order—IDs produced in earlier steps are required by later ones.

## Prerequisites

This skill assumes a button component already exists in the file (e.g., created via the `button-system` skill). If no button component exists, create a minimal one first: a frame named `Button/Variant=Primary, Size=Medium` converted to a component via `components({ method: "create", type: "from_node", ... })`.

You also need text styles for the title and body. Create them now if they don't exist:

```
styles({ method: "create", type: "text", name: "Text/Heading/SM",
  fontFamily: "Inter", fontWeight: 600, fontSize: 16, lineHeight: 24 })

styles({ method: "create", type: "text", name: "Text/Body/MD",
  fontFamily: "Inter", fontWeight: 400, fontSize: 14, lineHeight: 20 })
```

Record the returned `styleId` for each.

## Step 1: Create the Card Outer Frame

```
create_frame({
  name: "Card",
  layoutMode: "VERTICAL",
  primaryAxisAlignItems: "MIN",
  counterAxisAlignItems: "MIN",
  counterAxisSizingMode: "FIXED",
  width: 320,
  primaryAxisSizingMode: "AUTO",
  paddingLeft: 0, paddingRight: 0, paddingTop: 0, paddingBottom: 0,
  itemSpacing: 0,
  cornerRadius: 12,
  strokeWeight: 1,
  strokes: [{ type: "SOLID", color: "#E2E8F0" }],
  fills: [{ type: "SOLID", color: "#FFFFFF" }],
  clipsContent: true
})
```

Record the returned `cardFrameId`.

## Step 2: Create the Image Placeholder

Inside the card frame, create a rectangle acting as the image area:

```
create_frame({
  parentId: "<cardFrameId>",
  name: "Image Placeholder",
  layoutSizingHorizontal: "FILL",
  counterAxisSizingMode: "FIXED",
  height: 180,
  fills: [{ type: "SOLID", color: "#E2E8F0" }]
})
```

`layoutSizingHorizontal: "FILL"` makes it stretch to the card width automatically. Record the returned `imageFrameId`.

## Step 3: Create the Content Area

Create a vertical auto-layout frame inside the card for text and the CTA:

```
create_frame({
  parentId: "<cardFrameId>",
  name: "Content",
  layoutMode: "VERTICAL",
  primaryAxisAlignItems: "MIN",
  counterAxisAlignItems: "MIN",
  layoutSizingHorizontal: "FILL",
  primaryAxisSizingMode: "AUTO",
  paddingLeft: 16, paddingRight: 16, paddingTop: 16, paddingBottom: 16,
  itemSpacing: 12,
  fills: []
})
```

Record the returned `contentFrameId`.

## Step 4: Add Title Text

```
create_text({
  parentId: "<contentFrameId>",
  name: "Title",
  characters: "Card Title",
  textStyleId: "<Text/Heading/SM styleId>",
  layoutSizingHorizontal: "FILL",
  fills: [{ type: "SOLID", color: "#0F172A" }]
})
```

Record the returned `titleNodeId`.

## Step 5: Add Body Text

```
create_text({
  parentId: "<contentFrameId>",
  name: "Body",
  characters: "A short description that supports the card title and gives users context before they act.",
  textStyleId: "<Text/Body/MD styleId>",
  layoutSizingHorizontal: "FILL",
  fills: [{ type: "SOLID", color: "#475569" }]
})
```

Record the returned `bodyNodeId`.

## Step 6: Insert a Button Instance as the CTA

Find the primary medium button component. Use `get_node_info` on the file root or rely on a known `componentId` from the button-system skill. Then create an instance:

```
instances({
  method: "create",
  componentId: "<primary-medium-button-componentId>",
  parentId: "<contentFrameId>"
})
```

The instance inherits its width from the component. If you want the button to stretch full-width inside the content frame, patch it after creation:

```
patch_nodes({
  nodeId: "<button-instanceId>",
  layoutSizingHorizontal: "FILL"
})
```

## Step 7: Convert the Card to a Component

```
components({
  method: "create",
  type: "from_node",
  nodeId: "<cardFrameId>"
})
```

Record the returned `componentId`.

## Step 8: Expose Text Properties

Expose the title and body as component text properties so consumers can override them without detaching:

```
components({
  method: "create",
  type: "component_property",
  nodeId: "<componentId>",
  propertyName: "Title",
  propertyType: "TEXT",
  defaultValue: "Card Title",
  boundNodeId: "<titleNodeId>"
})

components({
  method: "create",
  type: "component_property",
  nodeId: "<componentId>",
  propertyName: "Body",
  propertyType: "TEXT",
  defaultValue: "A short description...",
  boundNodeId: "<bodyNodeId>"
})
```

## Step 9: Lint the Card Component

```
lint_node({
  nodeId: "<componentId>",
  rules: ["hardcoded-color", "no-text-style", "no-autolayout", "default-name"]
})
```

Common issues to watch for:
- `hardcoded-color` on the image placeholder fill—acceptable for placeholder, note it to the user
- `no-text-style` on title or body—means `textStyleId` was not applied; use `patch_nodes` to set the style
- `default-name` on the button instance—rename it via `patch_nodes({ nodeId: "<instanceId>", name: "CTA Button" })`

## Completion

Report to the user:
- The card `componentId`
- Component properties exposed: `Title`, `Body`
- Final structure: Card → [Image Placeholder, Content → [Title, Body, CTA Button]]
- Lint results and any issues resolved
