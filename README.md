# Clics

Privacy-friendly, cookieless web analytics — built for teams that want GDPR-friendly measurement without tracking cookies.

- **Website:** [clics.dev](https://clics.dev)
- **Dashboard:** [platform.clics.dev](https://platform.clics.dev)
- **Docs:** [clics.dev/docs](https://clics.dev/docs)

Clics measures traffic, goals, funnels, and sessions on your sites. This repository distributes **agent-facing tooling** so AI assistants can query analytics and manage your workspace from the terminal, a local MCP client, or a web-based assistant.

## What you get

| Tool             | Package / endpoint                                             | Best for                                                            |
| ---------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Clics plugin** | Available from Plugins                                         | ChatGPT and Codex                                                   |
| **Remote MCP**   | Hosted endpoint (see below)                                    | Claude.ai and other signed-in HTTP MCP clients                      |
| **Local MCP**    | [`@clicsdev/mcp`](https://www.npmjs.com/package/@clicsdev/mcp) | Cursor, Claude Code, VS Code, Windsurf, and other stdio MCP clients |
| **CLI**          | [`@clicsdev/cli`](https://www.npmjs.com/package/@clicsdev/cli) | Scripts, CI, and manual verification from the terminal              |
| **Agent skill**  | this repo                                                      | Teaching agents when and how to use Clics MCP + CLI                 |

Create an API key in the [dashboard](https://platform.clics.dev) for the local MCP or CLI.

---

## Remote MCP

Hosted [Model Context Protocol](https://modelcontextprotocol.io) server over HTTP Streamable. Use it with clients such as Claude.ai that support signed-in remote MCP connectors.

**Endpoint:**

```
https://api.clics.dev/v1/mcp
```

Compatible clients discover the sign-in flow automatically. Sign in to Clics and select the workspace you want to connect; no API key is required.

**Tools:** 18 core workspace tools: `list_projects`, `get_project`, `create_project`, `update_project`, `delete_project`, `list_goals`, `create_goal`, `update_goal`, `delete_goal`, `list_funnels`, `get_funnel`, `create_funnel`, `update_funnel`, `delete_funnel`, `list_sessions`, `get_session`, `list_session_events`, and `query_stats`.

### ChatGPT

1. Open **Plugins** in ChatGPT.
2. Search for **Clics** and click **Add**.
3. Click **Connect**, sign in to Clics, and select your workspace.
4. Start a conversation and select Clics when you want to use your analytics.

### Codex

1. Open **Plugins** in Codex.
2. Search for **Clics** and click **Add**.
3. Click **Connect**, sign in to Clics, and select your workspace.
4. Start a task and ask Codex to use Clics.

### Claude.ai

1. Go to **Settings** → **Connectors** → **Add custom connector**.
2. Enter `Clics` as the name and `https://api.clics.dev/v1/mcp` as the remote MCP server URL.
3. Leave the optional advanced fields empty and click **Add**.
4. Find Clics in the connectors list and click **Connect**.
5. Sign in to Clics, select your workspace, and enable the connector in a conversation.

### n8n

1. In your workflow, add an **AI Agent** node.
2. Add a tool → **MCP Client Tool**.
3. Configure:
   - **Endpoint:** `https://api.clics.dev/v1/mcp`
   - **Server Transport:** `HTTP Streamable`
   - **Authentication:** use the client’s sign-in option
4. Sign in to Clics, select your workspace, and choose which tools to expose.

---

## Local MCP (`@clicsdev/mcp`)

Local [Model Context Protocol](https://modelcontextprotocol.io) server (stdio) for cookie-free analytics. Manage projects, goals, funnels, sessions, and query stats from your editor when it can run a local process.

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

**Tools:** 22 tools: `list_projects`, `get_project`, `create_project`, `update_project`, `delete_project`; `list_goals`, `get_goal`, `get_goal_stats`, `create_goal`, `update_goal`, `delete_goal`; `list_funnels`, `get_funnel`, `get_funnel_stats`, `create_funnel`, `update_funnel`, `delete_funnel`; `list_session_filter_values`, `list_sessions`, `get_session`, `list_session_events`; and `query_stats`.

Goals support four types: `page`, `event`, `outbound`, and `scroll_depth`. The local MCP also returns dashboard-ready statistics for goals and funnels, and can discover the available values for Sessions filters. See the installed package README for each tool's complete input schema.

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

**Logout**

```bash
clics logout
```

**Examples**

```bash
clics projects list
clics goals list <project-id>
clics goals stats <goal-id> --date-range last30days --timezone UTC
clics sessions filter-values <project-id> --field country --date-range last30days
clics sessions list <project-id> --date-range last7days --limit 20
clics sessions get <project-id> <session-id>
clics sessions events <project-id> <session-id>
clics query <project-id> --metrics visitors pageviews bounce_rate --date-range last30days
```

Full command reference: [npm/@clicsdev/cli](https://www.npmjs.com/package/@clicsdev/cli)

---

## Agent skill

The `clics` skill teaches agents to prefer MCP when connected, fall back to CLI when not, and follow analytics playbooks (overview, top pages/countries, goals/funnels, sessions).

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
