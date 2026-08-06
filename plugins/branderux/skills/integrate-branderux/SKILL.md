---
name: integrate-branderux
description: Integrate BranderUX into a customer-facing AI agent (any framework — LangGraph, CrewAI, Mastra, Pydantic AI, Google ADK, Vercel AI SDK, provider SDKs or a custom loop) so its answers render as branded, interactive screens. Use when adding BranderUX/Brander to a site or agent product, wiring onQueryStream, or debugging an embed that renders text but no screens.
---

# Integrate BranderUX

BranderUX renders an agent's answers as branded, interactive UI. Your job: mount
`<Brander />` and wire the user's LLM into `onQueryStream` — correctly.

## Steps

1. `npm install @brander/sdk`.
2. Get credentials: a **project API key** (`bux_pk_…`, from the BranderUX dashboard →
   project → API Keys, or via the BranderUX MCP `create_api_key` tool) and the
   **projectId**. The key's allowed-origins must include the site's origin.
3. Mount:
   ```tsx
   import Brander, { sseStream } from "@brander/sdk";
   <Brander apiKey="bux_pk_..." projectId="..."
            onQueryStream={(params) => sseStream("/api/agent", { params })} />
   ```
4. Implement the backend. Ask the BranderUX MCP for the verified snippet:
   `get_integration_snippet` with `target` = `anthropic` | `openai` | `gemini` |
   `sse-backend` | `python` | `agent-framework` | `mcp-apps`. For framework specifics
   read the `agent-frameworks` doc (`read_doc`). Or follow the FOUR RULES below by hand.

## Which target

- **Provider SDK directly** (Anthropic/OpenAI/Gemini) → use the matching snippet; the
  SDK's `anthropicStream`/`openaiStream`/`geminiStream` adapters emit AG-UI for you.
- **An agent framework** (LangGraph, CrewAI, Mastra, Pydantic AI, Google ADK, AWS
  Strands, LlamaIndex, Agno, AG2, Microsoft Agent Framework, Vercel AI SDK, n8n, or
  homegrown) → `agent-framework`. Your route unwraps `params`, runs the agent, streams
  AG-UI. Do NOT point the SDK at a framework's own AG-UI endpoint — those expect AG-UI's
  `RunAgentInput` body, not `{ params }`, and the mismatch fails silently.
- **The product IS an MCP server** → `mcp-apps`: `@brander/mcp-tools` renders branded
  screens inline instead of the SDK.

## The four rules (every silent failure is one of these)

1. **Forward `params.system`.** It carries ALL screen-generation instructions; in the
   default (flexible) mode dropping it means plain text forever, no error. Anthropic:
   `system: params.system` · OpenAI: prepend a `role:"system"` message · Gemini:
   `systemInstruction`.
2. **`params.tools` is optional.** Always `params.tools?.anthropic` — it is absent in
   flexible mode and unguarded access crashes at runtime. Gemini needs
   `[{ functionDeclarations: params.tools.gemini }]`.
3. **The body is nested.** `sseStream` POSTs `{ params }` — backends read
   `req.body.params`, never `req.body.messages`.
4. **End with RUN_FINISHED.** SSE frames are `data: <AG-UI event JSON>\n\n`; a stream
   that closes without a RUN_FINISHED event leaves the UI loading forever. Provider
   adapters (`anthropicStream` etc.) handle this when used.

Also: provider API keys live server-side only — `NEXT_PUBLIC_*` keys are leaked to every
visitor.

## Verify

Run the app, send a query, and confirm (a) a screen (not just text) renders, (b) clicking
an element fires a follow-up query, (c) the browser console has no Brander errors. If
screens don't render, rule 1 or 3 is broken — check the backend receives `params.system`
and forwards it.
