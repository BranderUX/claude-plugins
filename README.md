# BranderUX Claude plugins

Official Claude Code plugin for building on [BranderUX](https://branderux.com): build agentic
applications, in minutes. One message in, your full agentic application out: website and MCP
app, live. Companies with their own agent connect it as a first-class pillar.

## Install

```
/plugin marketplace add BranderUX/claude-plugins
/plugin install branderux@branderux
```

Installing the plugin also wires up the **BranderUX MCP** (`mcp.branderux.com/mcp`): on
first use your browser opens for a one-click BranderUX sign-in (beta accounts connect today;
everyone else lands on the waiting list). No API keys to paste.

## What's inside

| Skill | What it does |
|---|---|
| `integrate-branderux` | Wire BranderUX into a customer-facing agent, any framework (LangGraph, CrewAI, Mastra, Pydantic AI, ADK, provider SDKs, custom), with the four rules that make integrations work first try |
| `branderux-agentic-site` | Build a FULL agentic site (like [nova.branderux.app](https://nova.branderux.app)): project, elements, screens, agent endpoint |
| `branderux-custom-element` | Author sandboxed custom elements that behave correctly in production |

Plus the MCP server: 42 tools to create and control projects, brand settings, custom
elements, screens and API keys, and to build the hosted agent itself (agent config, business
data, connectors, skills, the designed home screen, publish), with knowledge tools
(`get_started`, `read_doc`, `search_docs`, `get_integration_snippet`), the reference docs as
MCP resources, and guided prompts (`build-agentic-site`, `integrate-branderux`,
`author-custom-element`). In clients with MCP Apps support, published custom elements render
live in the panel (`preview_element`). Destructive tools are annotated and require explicit
confirmation.

## License

MIT
