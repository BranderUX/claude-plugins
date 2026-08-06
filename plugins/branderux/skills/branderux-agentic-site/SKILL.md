---
name: branderux-agentic-site
description: Build a FULL agentic application on BranderUX — a site that is one full-screen Brander surface where every screen is generated at runtime (like nova.branderux.app). Use when the user wants an AI-native site, storefront, dashboard or portal built on their agent, not just an embedded widget.
---

# Build a full agentic site on BranderUX

The end state: the customer's site has NO pages — one `<Brander isFullscreen />` surface +
one agent endpoint. Home, listings, detail views, checkout-style flows: all generated.
Reference implementation: nova.branderux.app (github.com/BranderUX/atelier-nova).

Work through the BranderUX MCP tools (sign-in happens in the browser on first use).
Call `get_started` first, and `read_doc` for every wire format before writing it.

## Phase 1 — Project

1. `whoami` → then `create_project` with the brand name.
2. `update_brand_settings` — colors, fonts, radius from the customer's brand.
3. `update_project_settings` — `{"uiGenerationMode": "flexible"}`; add `customPages` for
   nav entries (e.g. store sections) if wanted.

## Phase 2 — The world (the agent's knowledge)

The customer's agent endpoint owns the domain "world": catalog/data, user context,
behavior rules. Structure the system prompt as: identity → the signed-in user + their
history → the catalog/data as JSON (exact ids, prices, image URLs) → behavior rules →
voice. Two proven patterns:
- **Canned deterministic intents**: answer known queries (the home page) with a pre-built
  A2UI block, zero LLM tokens, instant paint (`read_doc a2ui-wire-format`).
- **Format guard + retry**: require a UI block in EVERY reply; conversation history
  summaries teach the model to answer text-only otherwise. Server-side: buffer until the
  A2UI marker, silently retry once if a reply has no block.

## Phase 3 — Elements (5–10, big blocks)

One element per screen AREA (hero, product grid, order panel, answer card, suggestions
row…), not per widget. Write the TSX yourself following `read_doc
custom-elements-contract`, publish with `create_element` (it pre-flight validates).
Product images: absolute https URLs from the customer's assets.

## Phase 4 — Screens (the pattern library)

5–10 example screens via `put_screen` (`read_doc screens-wire-format` first — 0-based
positions, pinned element versions). Each screen teaches a pattern: home, listing,
answer+action side-by-side, confirmation. The runtime AI composes new screens from these.

## Phase 5 — The site shell

A minimal Next.js app: one page mounting `<Brander variant="chat" isFullscreen />`
client-side, one `/api/agent/stream` route (LLM + the world prompt + `params.system`
appended), and `create_api_key` for the embed key (allowed-origins = the site's origin).
Use the `integrate-branderux` skill's four rules for the endpoint.

## Verify (headless, before demoing)

Playwright: load the site, wait ~30s for the generated home, screenshot; send a domain
query, click an element, confirm the follow-up screen. Test EVERY demo moment the same
way. Common traps: breakpoints resolve against the element iframe (use `sm` keys);
popovers must not move focus; view swaps must keep height; motion needs container room.
