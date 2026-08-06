---
name: branderux-custom-element
description: Author a BranderUX custom element (sandboxed TSX component) correctly — props contract, callbacks, sandbox constraints — and publish it via the BranderUX MCP. Use when creating or fixing a custom element, or when an element renders but interactions/hover/popovers behave badly.
---

# Author a BranderUX custom element

Elements are TSX components rendered in a SANDBOXED IFRAME on every surface. You write
the code; `create_element` / `publish_element_version` validate and publish it. Full
contract: `read_doc custom-elements-contract` on the BranderUX MCP.

## The contract in one glance

```tsx
import { useState, useEffect } from "react";
import { Box, Typography, Button } from "@mui/material";

export interface Props {                      // exactly this export
  items: { id: string; name: string; price: number; imageUrl: string }[];
  onSelectItem?: (item: { id: string; name: string }) => void;
  onItemContextMenu?: (event: React.MouseEvent, item: unknown) => void;
}

export default function Component({ items, onSelectItem, onItemContextMenu }: Props) {
  // no prop defaults; demo data lives in defaultProps
}
```

Allowed imports: react, @mui/material, @mui/system, @emotion/react, @emotion/styled,
lucide-react, recharts, framer-motion, date-fns. Nothing else resolves in the sandbox.

## Interactivity

- External action (sends an agent query) = optional, action-named callback invoked with
  optional chaining: `onAddToCart?.(item)`. One callback per gesture.
- Internal state (selection, tabs, steps) = `useState`, resynced via `useEffect` when the
  seed prop changes.
- Right-click: `onItemContextMenu` wired per item on the same node as its onClick —
  `onContextMenu={(e) => { e.preventDefault(); onItemContextMenu?.(e, item); }}`.
- `clickQueryTemplate` maps clicks to queries: plain string for the primary action or a
  JSON map `{"$primary": "...", "onAddToCart": "Add {name} to my cart"}` — `{tokens}`
  resolve against the callback payload.

## Sandbox constraints (the four production-bug classes)

1. **Popovers never move focus**: every Select/Menu/Dialog gets `autoFocus: false` +
   `disableAutoFocusItem/disableAutoFocus/disableEnforceFocus/disableRestoreFocus/
   disableScrollLock: true` — otherwise the HOST page scroll-jumps.
2. **Motion needs room**: hover lift/scale/shadow clips at the iframe bounds — pad the
   root, or keep the motion inside an overflow-hidden card.
3. **View swaps keep one height**: in-place step changes (order → confirm) must not change
   the component's height, or the content-sized iframe resizes and the host layout jumps.
4. **Breakpoints are iframe-relative**: an element in a half-width slot never reaches
   `md` — use `sm` keys for responsive behavior inside slots.

Also ship a skeleton (`export default function SkeletonComponent()`, Box/Skeleton/Stack
only) and realistic `defaultProps` that make the element demo well with zero data.

## Verify visually

After publishing, call `preview_element` — in clients with MCP Apps support the element
renders live in the panel with its demo props, and every callback shows the exact query
the click would send. Use it to confirm wiring before placing the element on a screen.
