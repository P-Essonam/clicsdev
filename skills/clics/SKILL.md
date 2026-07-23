---
name: clics
description: Query Clics analytics and manage projects, goals, and funnels through the Clics MCP server (`@clicsdev/mcp`), with the `clics` CLI (`@clicsdev/cli`) as a fallback. Use when a user asks about their analytics, mentions Clics, works with the Clics toolset, verifies Clics manually, or needs to choose between MCP and CLI.
---

# Clics

Clics is privacy-friendly, cookieless web analytics. Agents manage projects, goals, funnels, and analytics stats in two ways:

- **MCP tools** (`clics` server via `@clicsdev/mcp`) — preferred for agents.
- **CLI** (`@clicsdev/cli`, binary `clics`) — fallback when MCP is unavailable, for manual verification, or for scripts and CI.

## Choosing MCP vs CLI

Prefer **MCP tools** whenever the `clics` MCP server is connected — no shell is required, and the API key is supplied by the MCP client environment.

Use the **CLI** when:

- The MCP server is not connected in this session.
- You need to verify behavior manually or from a script or terminal.
- You are automating in CI where stdio MCP is not available.

## Authentication

**MCP** — set in the MCP client config (never pass the API key as a tool argument):

```bash
CLICS_API_KEY=your_api_key
```

Typical Cursor setup: `npx -y @clicsdev/mcp` with that environment variable.

**CLI** — one-time local configuration (stored in `~/.config/clics/`):

```bash
npm install -g @clicsdev/cli
# or: npx @clicsdev/cli …
clics init --api-key "<your-api-key>"
```

Re-run `clics init` to rotate the key. Clear stored credentials with `clics logout`. Successful CLI commands print a single JSON document on stdout; errors are written to stderr with a non-zero exit code.

```bash
clics logout
```

## MCP tools (preferred)

Server name: `clics`. Call tools directly; full input schemas are defined on each tool.

### Projects

| Tool | Key inputs |
|------|------------|
| `list_projects` | optional `cursor`, `limit` |
| `get_project` | `project_id` |
| `create_project` | `name`, `website_url`, optional `allow_localhost` |
| `update_project` | `project_id`, optional `name`, `website_url`, `allow_localhost` |
| `delete_project` | `project_id` |

### Goals

`env_id`: `production` \| `development` (default `production`).

| Tool | Key inputs |
|------|------------|
| `list_goals` | `project_id`, optional `env_id` |
| `create_goal` | `project_id`, `goal_type` (`page`\|`event`), `display_name`, optional `env_id`, `page_path`, `event_name` |
| `update_goal` | `goal_id`, `goal_type`, `display_name`, optional `page_path`, `event_name` |
| `delete_goal` | `goal_id` |

### Funnels

| Tool | Key inputs |
|------|------------|
| `list_funnels` | `project_id`, optional `env_id`, `cursor`, `limit` |
| `get_funnel` | `funnel_id` |
| `create_funnel` | `project_id` + body (`name`, `conversion_window`, `steps`, optional `env_id`) |
| `update_funnel` | `funnel_id` + body without `env_id` |
| `delete_funnel` | `funnel_id` |

### Stats

| Tool | Key inputs |
|------|------------|
| `query_stats` | full Stats body: `project_id`, `metrics`, `date_range`, optional `domain`, `dimensions`, `filters`, `order_by`, `include`, `pagination` |

MCP example (`query_stats` input):

```json
{
  "project_id": "your_project_id",
  "metrics": ["visitors", "pageviews", "bounce_rate"],
  "date_range": "last30days",
  "include": { "previous_period": true }
}
```

Responses return JSON text in `content` and the same object in `structuredContent`. Failures set `isError: true`.

## CLI reference (fallback)

Package: `@clicsdev/cli`. Binary: `clics`. Help: `clics --help`, `clics --version`, `clics help <command>`.

Run without a global install: `npx @clicsdev/cli <command> …`.

### Projects

**List**

```bash
clics projects list
clics projects list --limit 20
clics projects list --cursor "<cursor>"
```

**Get**

```bash
clics projects get <project-id>
```

**Create**

```bash
clics projects create --name "My site" --website-url https://example.com
clics projects create --name "My site" --website-url https://example.com --allow-localhost
```

**Update**

```bash
clics projects update <project-id> --name "New name"
clics projects update <project-id> --website-url https://new.example.com
clics projects update <project-id> --allow-localhost
```

**Delete**

```bash
clics projects delete <project-id>
```

### Goals

`env_id` is `production` or `development` (API default: `production`).

**List**

```bash
clics goals list <project-id>
clics goals list <project-id> --env-id development
```

**Create** — page goal:

```bash
clics goals create <project-id> --goal-type page --display-name "Signup page" --page-path /signup
```

**Create** — event goal:

```bash
clics goals create <project-id> --goal-type event --display-name "Purchase" --event-name purchase
```

Optional environment:

```bash
clics goals create <project-id> --goal-type page --display-name "Home" --page-path / --env-id development
```

**Update**

```bash
clics goals update <goal-id> --goal-type page --display-name "Signup" --page-path /signup
clics goals update <goal-id> --goal-type event --display-name "Purchase" --event-name purchase
```

**Delete**

```bash
clics goals delete <goal-id>
```

### Funnels

List, get, and delete use flags. Create and update take JSON via `--body` (inline or `@file.json`).

**List**

```bash
clics funnels list <project-id>
clics funnels list <project-id> --env-id production --limit 20
clics funnels list <project-id> --cursor "<cursor>"
```

**Get**

```bash
clics funnels get <funnel-id>
```

**Create**

```bash
clics funnels create <project-id> --body @funnel.json
```

Example `funnel.json`:

```json
{
  "name": "Signup funnel",
  "conversion_window": { "value": 7, "unit": "days" },
  "steps": [
    {
      "name": "Landing",
      "filters": [{ "filter_type": "page", "operator": "is", "values": ["/"] }]
    },
    {
      "name": "Signup",
      "filters": [{ "filter_type": "page", "operator": "is", "values": ["/signup"] }]
    }
  ]
}
```

Optional `env_id` in the body: `production` | `development`.

**Update** (same shape as create, without `env_id`):

```bash
clics funnels update <funnel-id> --body @funnel-update.json
```

**Delete**

```bash
clics funnels delete <funnel-id>
```

### Query analytics

`--metrics` and `--date-range` are required (unless using `--file`).

**Presets**

```bash
clics query <project-id> --metrics visitors --date-range last24h
clics query <project-id> --metrics visitors --date-range last7days
clics query <project-id> --metrics visitors --date-range last30days
clics query <project-id> --metrics visitors --date-range last3months
clics query <project-id> --metrics visitors --date-range last12months
clics query <project-id> --metrics visitors --date-range monthToDate
clics query <project-id> --metrics visitors --date-range quarterToDate
clics query <project-id> --metrics visitors --date-range yearToDate
clics query <project-id> --metrics visitors --date-range allTime
```

**Basic KPIs**

```bash
clics query <project-id> --metrics visitors pageviews bounce_rate --date-range last30days
```

**Comparison / totals**

```bash
clics query <project-id> --metrics visitors pageviews bounce_rate --date-range last30days --previous-period --total-rows
```

**Domain filter**

```bash
clics query <project-id> --metrics visitors --date-range last7days --domain example.com
```

Use `--domain localhost` for development traffic only.

**Breakdown**

```bash
clics query <project-id> --metrics visitors pageviews --date-range last30days --dimensions visit:country
```

**Pagination**

```bash
clics query <project-id> --metrics visitors --date-range last30days --dimensions visit:country --limit 50 --offset 0
```

**Advanced file** (filters, `order_by`, custom date ranges):

```bash
clics query <project-id> --file query.json
```

Example `query.json`:

```json
{
  "metrics": ["visitors", "pageviews", "bounce_rate"],
  "date_range": "last30days",
  "dimensions": ["visit:country"],
  "filters": [["is", "visit:country", ["US", "FR"]]],
  "order_by": [["visitors", "desc"]],
  "include": {
    "previous_period": true,
    "total_rows": true
  },
  "pagination": {
    "limit": 50,
    "offset": 0
  }
}
```

`project_id` in the file is overwritten by the CLI argument.

**Allowed metrics:** `visitors` · `visits` · `pageviews` · `bounce_rate` · `visit_duration` · `views_per_visit` · `conversion_rate` · `events`

- KPI / time-series: `visitors`, `visits`, `pageviews`, `bounce_rate`, `visit_duration`, `views_per_visit`
- Breakdown: `visitors`, `pageviews`, `conversion_rate`, `events`

**Allowed date ranges:** `last24h` · `last7days` · `last30days` · `last3months` · `last12months` · `monthToDate` · `quarterToDate` · `yearToDate` · `allTime` (custom ISO pairs via `--file`)

**Allowed dimensions:** `event:page` · `event:hostname` · `visit:country` · `visit:device` · `visit:browser` · `visit:os` · `visit:referrer` · `visit:utm_source` · `visit:utm_medium` · `visit:utm_campaign` · `visit:utm_term` · `visit:utm_content` · `time` · `time:hour` · `time:day`

### Scripting

```bash
clics projects list | jq '.projects[].id'
```

Prefer `--body @file.json` / `--file file.json` over inline JSON (especially on PowerShell).

## Playbooks

### 1. Overview (last 30 days)

1. Run `list_projects` or `clics projects list` and pick a `project_id`.
2. Run `query_stats` or `clics query <id> --metrics visitors visits pageviews bounce_rate visit_duration views_per_visit --date-range last30days --previous-period`.
3. Summarize KPIs and period-over-period changes from the JSON response.

### 2. Top pages and countries

1. Resolve `project_id` as above.
2. **Pages:** `query_stats` with `metrics: ["visitors","pageviews"]`, `date_range: "last30days"`, `dimensions: ["event:page"]`, `order_by: [["visitors","desc"]]`, `pagination: { "limit": 10 }`.  
   CLI: `clics query <id> --metrics visitors pageviews --date-range last30days --dimensions event:page --limit 10`
3. **Countries:** same query with `dimensions: ["visit:country"]`.  
   CLI: `clics query <id> --metrics visitors pageviews --date-range last30days --dimensions visit:country --limit 10`

### 3. Goals and funnels check

1. Run `list_goals` or `clics goals list <project-id>` (add `--env-id development` if needed).
2. Run `list_funnels` or `clics funnels list <project-id>`.
3. Optionally run `get_funnel` or `clics funnels get <funnel-id>` for step details.
4. Report what is configured. Do not create or delete resources unless the user asked.

## Manual verification

Confirm authentication and API access with the CLI so JSON is visible in the terminal:

```bash
clics projects list
clics query <project-id> --metrics visitors --date-range last7days
```

Expect a JSON projects payload and a stats `results` array. Common errors:

- `API key is required. Run clics init.` — run `clics init --api-key "…"`
- `401` / `invalid key` — rotate or recreate the key, then run `clics init` again
- `403` / `UPGRADE_REQUIRED` — a paid plan is required for API access

## Links

- Product: https://clics.dev
- Dashboard: https://platform.clics.dev
- Docs: https://clics.dev/docs
- MCP: `@clicsdev/mcp`
- CLI: `@clicsdev/cli`
