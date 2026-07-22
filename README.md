# Clics

Privacy-friendly, cookieless web analytics — built for teams that want GDPR-friendly measurement without tracking cookies.

- **Website:** [clics.dev](https://clics.dev)
- **Dashboard:** [platform.clics.dev](https://platform.clics.dev)
- **Docs:** [clics.dev/docs](https://clics.dev/docs)

Clics measures traffic, goals, and funnels on your sites. This repository distributes **agent-facing tooling** so AI assistants can query analytics and manage your workspace from the terminal, a local MCP client, or a web-based assistant.

## What you get

| Tool | Package / endpoint | Best for |
|------|-------------------|----------|
| **Remote MCP** | Hosted endpoint (see below) | ChatGPT, Claude.ai, n8n, and other HTTP MCP clients |
| **Local MCP** | [`@clicsdev/mcp`](https://www.npmjs.com/package/@clicsdev/mcp) | Cursor, Claude Code, VS Code, Windsurf, and other stdio MCP clients |
| **CLI** | [`@clicsdev/cli`](https://www.npmjs.com/package/@clicsdev/cli) | Scripts, CI, and manual verification from the terminal |
| **Agent skill** | this repo | Teaching agents when and how to use Clics MCP + CLI |

Create an API key in the [dashboard](https://platform.clics.dev).

---

## Remote MCP

Hosted [Model Context Protocol](https://modelcontextprotocol.io) server over HTTP Streamable. Use this when your assistant runs in the browser or in a workflow tool and cannot run a local `npx` process.

**Endpoint** (replace `YOUR_API_KEY` with your Clics API key):

```
https://api.clics.dev/YOUR_API_KEY/v1/mcp
```

The API key is embedded in the URL (same pattern as [Firecrawl remote MCP](https://docs.firecrawl.dev/mcp-server)). Set authentication to **None** in the client; no extra headers are required.

**Tools:** same 15 tools as local MCP (`list_projects`, `get_project`, `create_project`, `update_project`, `delete_project`, `list_goals`, `create_goal`, `update_goal`, `delete_goal`, `list_funnels`, `get_funnel`, `create_funnel`, `update_funnel`, `delete_funnel`, `query_stats`).

### ChatGPT

1. Create an API key in the [dashboard](https://platform.clics.dev).
2. In ChatGPT, open **Settings** → **Advanced Settings** and enable **Developer mode**.
3. Open **Apps & Connectors** → **Create**.
4. Fill in:
   - **Name:** `Clics MCP`
   - **MCP Server URL:** `https://api.clics.dev/YOUR_API_KEY/v1/mcp`
   - **Authentication:** `None`
5. Save and enable the connector in a conversation.

See also: [Firecrawl ChatGPT MCP guide](https://docs.firecrawl.dev/developer-guides/mcp-setup-guides/chatgpt) for the same connector flow.

### Claude.ai

1. Create an API key in the [dashboard](https://platform.clics.dev).
2. Go to **Settings** → **Connectors** → **Add custom connector**.
3. Set **URL** to `https://api.clics.dev/YOUR_API_KEY/v1/mcp` (leave OAuth fields blank).
4. In a conversation, enable the connector from **Connectors**.

See also: [Firecrawl Claude.ai MCP guide](https://docs.firecrawl.dev/developer-guides/mcp-setup-guides/claude-ai).

### n8n

1. Create an API key in the [dashboard](https://platform.clics.dev).
2. In your workflow, add an **AI Agent** node.
3. Add a tool → **MCP Client Tool**.
4. Configure:
   - **Endpoint:** `https://api.clics.dev/YOUR_API_KEY/v1/mcp`
   - **Server Transport:** `HTTP Streamable`
   - **Authentication:** `None`
5. Choose which tools to expose (All, Selected, or All Except).

See also: [Firecrawl n8n setup](https://docs.firecrawl.dev/mcp-server#running-on-n8n) for the same MCP Client Tool flow.

---

## Local MCP (`@clicsdev/mcp`)

Local [Model Context Protocol](https://modelcontextprotocol.io) server (stdio) for cookie-free analytics. Manage projects, goals, funnels, and query stats from your editor when it can run a local process.

**Install**

```bash
npm install -g @clicsdev/mcp
# or: npx -y @clicsdev/mcp
```

**Authenticate** — set `CLICS_API_KEY` in your MCP client config (never pass the key as a tool argument).

**Cursor** (`.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "clics": {
      "command": "npx",
      "args": ["-y", "@clicsdev/mcp"],
      "env": {
        "CLICS_API_KEY": "your_api_key"
      }
    }
  }
}
```

**Tools:** `list_projects`, `get_project`, `create_project`, `update_project`, `delete_project`, `list_goals`, `create_goal`, `update_goal`, `delete_goal`, `list_funnels`, `get_funnel`, `create_funnel`, `update_funnel`, `delete_funnel`, `query_stats`.

Full client setup: [npm/@clicsdev/mcp](https://www.npmjs.com/package/@clicsdev/mcp)

---

## CLI (`@clicsdev/cli`)

Command-line interface for the same operations — JSON on stdout, ideal for automation and quick checks.

**Install**

```bash
npm install -g @clicsdev/cli
# or: npx @clicsdev/cli --help
```

**Authenticate**

```bash
clics init --api-key "<your-api-key>"
```

**Examples**

```bash
clics projects list
clics goals list <project-id>
clics query <project-id> --metrics visitors pageviews bounce_rate --date-range last30days
```

Full command reference: [npm/@clicsdev/cli](https://www.npmjs.com/package/@clicsdev/cli)

---

## Agent skill

The `clics` skill teaches agents to prefer MCP when connected, fall back to CLI when not, and follow analytics playbooks (overview, top pages/countries, goals/funnels).

**Install** with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add P-Essonam/clicsdev --skill clics
```

Global install (all projects):

```bash
npx skills add P-Essonam/clicsdev --skill clics -g -y
```

List skills without installing:

```bash
npx skills add P-Essonam/clicsdev --list
```

Skill source: [`skills/clics/SKILL.md`](skills/clics/SKILL.md)

---

## License

MIT
