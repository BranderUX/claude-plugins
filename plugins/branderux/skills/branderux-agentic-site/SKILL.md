---
name: branderux-agentic-site
description: Build a FULL agentic application on BranderUX — a site that is one full-screen Brander surface where every screen is generated at runtime (like nova.branderux.app). Use when the user wants an AI-native site, storefront, dashboard or portal built on their agent, not just an embedded widget.
---

# Build a full agentic site on BranderUX

The end state: the customer's site has NO pages — one `<Brander isFullscreen />` surface,
answered by the agent BranderUX hosts for the project (or by the customer's own endpoint).
Home, listings, detail views, checkout-style flows: all generated.
Reference implementation: nova.branderux.app (github.com/BranderUX/atelier-nova), which runs
on the hosted agent.

Work through the BranderUX MCP tools (sign-in happens in the browser on first use).
Call `get_started` first, and `read_doc` for every wire format before writing it.

## Phase 1 — Project

1. `whoami` → then `create_project` with the brand name.
2. `update_brand_settings` — colors, fonts, radius from the customer's brand.
3. `update_project_settings` — `{"uiGenerationMode": "flexible"}`; add `customPages` for
   nav entries (e.g. store sections) if wanted.

## Phase 2 — The world (the agent's knowledge)

Pick the answering agent up front. **Hosted (default, the Atelier Nova pattern):** the
project's own agent, built with the hosted-agent tools (`read_doc hosted-agent-contract`
first, its owner questions included):
- `define_entity` + `seed_records` for the catalog/data (every field described; scalars for
  anything filterable; images as absolute https URLs) and for the records visitors create
  (orders, bookings): `writePolicy: "open"`, and `policies.writePolicies["create_<entity>"]`
  = `"auto"` when the site is an SDK embed on the customer's own host, where the platform's
  confirm card cannot render and the visitor's click is the confirmation.
- `upsert_agent_config` with the persona: identity → the signed-in user + their history →
  the query recipes per data tool → the clock (dates from the platform's clock block) →
  assets that are not records → the write flow → behavior rules → voice. The data stays in
  entities the agent queries; never paste it into the persona. Always `policies.language`
  and `policies.timezone`.
- `set_home_screen` for the landing page: the first custom page's query, verbatim, replayed
  with zero model calls (see Phase 5).

**Own agent:** the customer's endpoint owns the world. Structure its system prompt as:
identity → the signed-in user + their history → the catalog/data as JSON (exact ids,
prices, image URLs) → behavior rules → voice. Two proven patterns: canned deterministic
intents (a pre-built A2UI block for the home, `read_doc a2ui-wire-format`) and a format
guard + retry (require a UI block in EVERY reply; buffer until the A2UI marker, retry once).

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

A minimal Next.js app: one page mounting `<Brander apiKey projectId variant="chat"
isFullscreen />` client-side, with NO handler (the hosted agent answers), and
`create_api_key` for the embed key (allowed-origins = the site's exact origin, plus
`http://localhost:<port>` while developing). The landing is the designed home from
`set_home_screen`: `matchQuery` equal to the first custom page's query, `screenId` an
existing custom screen, static `data` keyed by placement id.

Own-agent variant: add `onQueryStream` and one `/api/agent/stream` route (LLM + the world
prompt + `params.system` appended); use the `integrate-branderux` skill's four rules for
the endpoint.

## Verify (headless, before demoing)

Playwright: load the site, wait ~30s for the generated home, screenshot; send a domain
query, click an element, confirm the follow-up screen. Test EVERY demo moment the same
way. Common traps: breakpoints resolve against the element iframe (use `sm` keys);
popovers must not move focus; view swaps must keep height; motion needs container room.
