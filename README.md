# Clics

Privacy-friendly, cookieless web analytics — built for teams that want GDPR-friendly measurement without tracking cookies.

- **Website:** [clics.dev](https://clics.dev)
- **Dashboard:** [platform.clics.dev](https://platform.clics.dev)
- **Docs:** [clics.dev/docs](https://clics.dev/docs)

Clics measures traffic, goals, and funnels on your sites. This repository distributes **agent-facing tooling** so AI assistants can query analytics and manage your workspace from the terminal or an MCP client.

## What you get

| Tool | Package | Best for |
|------|---------|----------|
| **MCP server** | [`@clicsdev/mcp`](https://www.npmjs.com/package/@clicsdev/mcp) | Cursor, Claude Code, VS Code, Windsurf, and any MCP client |
| **CLI** | [`@clicsdev/cli`](https://www.npmjs.com/package/@clicsdev/cli) | Scripts, CI, and manual verification from the terminal |
| **Agent skill** | this repo | Teaching agents when and how to use Clics MCP + CLI |

---

## MCP (`@clicsdev/mcp`)

Local [Model Context Protocol](https://modelcontextprotocol.io) server for cookie-free analytics. Manage projects, goals, funnels, and query stats from your AI editor.

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
