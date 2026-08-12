---
name: clics
description: Query Clics analytics and manage projects, goals, funnels, and sessions through a connected Clics MCP server, with the `clics` CLI (`@clicsdev/cli`) as a fallback. Use when a user asks about website analytics, mentions Clics, works with the Clics toolset, or needs to inspect traffic, conversions, or visitor journeys.
---

# Clics

Clics is privacy-friendly, cookieless web analytics. Agents manage projects, goals, funnels, sessions, and analytics stats in three ways:

- **Hosted MCP tools** (`clics` server) — preferred when installed through a plugin or remote connector.
- **Local MCP tools** (`@clicsdev/mcp`) — for clients configured to run a local process.
- **CLI** (`@clicsdev/cli`, binary `clics`) — fallback when MCP is unavailable, for manual verification, or for scripts and CI.

## Choosing MCP vs CLI

Prefer **MCP tools** whenever the `clics` MCP server is connected. No shell is required. Hosted connections authenticate through the Clics sign-in flow; local connections receive an API key from the MCP client environment.

Use the **CLI** when:

- The MCP server is not connected in this session.
- You need to verify behavior manually or from a script or terminal.
- You are automating in CI where stdio MCP is not available.

## Authentication

**Hosted MCP** — connect and sign in when prompted. Do not ask the user for an API key.

**Local MCP** — set the API key in the MCP client config (never pass it as a tool argument):

```bash
CLICS_API_KEY=your_api_key
```

Typical local setup: `npx -y @clicsdev/mcp` with that environment variable.

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

Server name: `clics`. Call tools directly; full input schemas are defined on each tool. Confirm the connected server's tool list before calling it. The local `@clicsdev/mcp` server exposes the complete set below; if a connected MCP server does not offer a read operation, use the CLI rather than inventing a direct request.

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
| `get_goal` | `goal_id` |
| `get_goal_stats` | `goal_id`, optional `domain`, period, `timezone` (UTC by default), `referrer_ai_provider` |
| `create_goal` | `project_id`, `goal_type` (`page`\|`event`\|`outbound`\|`scroll_depth`), `display_name`, `rule`, optional `env_id` |
| `update_goal` | `goal_id`, optional `display_name`, `goal_type`, or partial `rule` |
| `delete_goal` | `goal_id` |

Use the rule that matches `goal_type`:

| Goal type | Required rule |
|-----------|---------------|
| `page` | `{ "page_path": "/signup" }` |
| `event` | `{ "event_name": "purchase" }` |
| `outbound` | `{ "outbound_url": "https://partner.example.com/signup" }` |
| `scroll_depth` | `{ "page_path": "/pricing", "scroll_depth_threshold": 75 }` |

Use an exact HTTP(S) URL for an outbound goal. Use a whole scroll-depth percentage from `1` to `100`. When changing a goal's type, send the complete rule for the new type. `get_goal_stats` returns dashboard-ready `totals`, `comparison`, `series`, and `meta.timezone` for every goal type.

### Funnels

| Tool | Key inputs |
|------|------------|
| `list_funnels` | `project_id`, optional `env_id`, `cursor`, `limit` |
| `get_funnel` | `funnel_id` |
| `get_funnel_stats` | `funnel_id`, optional `domain`, period, `timezone` (UTC by default), `referrer_ai_provider` |
| `create_funnel` | `project_id` + body (`name`, `conversion_window`, `steps`, optional `env_id`) |
| `update_funnel` | `funnel_id` + body without `env_id` |
| `delete_funnel` | `funnel_id` |

### Sessions

Period presets match `query_stats` (`last24h`, `last7days`, …, `allTime`). For a custom range, pass both `start` and `end`. Pass an IANA `timezone` when calendar boundaries must use a local timezone; UTC is the default. Use optional `domain` to scope a site.

| Tool | Key inputs |
|------|------------|
| `list_session_filter_values` | `project_id`, `field`, optional scope, `timezone`, `limit`; fields: `country`, `device`, `browser`, `os`, `page_entry`, `page_exit`, `referrer` |
| `list_sessions` | `project_id`, optional `domain`, `date_range` (default `last7days`), `start`, `end`, `timezone`, `cursor`, `limit` |
| `get_session` | `project_id`, `session_id`, optional `domain`, `date_range` (default `allTime`), `start`, `end`, `timezone` |
| `list_session_events` | `project_id`, `session_id`, optional `domain`, `date_range` (default `allTime`), `start`, `end`, `timezone` |

### Stats

| Tool | Key inputs |
|------|------------|
| `query_stats` | full Stats body: `project_id`, `metrics`, `date_range`, optional `domain`, `timezone` (UTC by default), `dimensions`, `filters`, `order_by`, `include`, `pagination` |

```json
{
  "project_id": "your_project_id",
  "metrics": ["visitors", "pageviews", "bounce_rate"],
  "date_range": "last30days",
  "timezone": "UTC",
  "include": { "previous_period": true }
}
```

For `query_stats`, use one time dimension (`time`, `time:hour`, or `time:day`) or up to two breakdown dimensions. `event:name`, `event:outbound_url`, and `referrer:ai_provider` support event and AI analytics. Filters additionally support `event:goal`. For a custom stats range, set `date_range` to a two-item ISO date pair.

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

**Create** â€” outbound-link and scroll-depth goals:

```bash
clics goals create <project-id> --goal-type outbound --display-name "Documentation" --outbound-url https://example.com/docs
clics goals create <project-id> --goal-type scroll_depth --display-name "Article 75%" --page-path /article --scroll-depth-threshold 75
```

Goal rules are type-specific: `page` requires `--page-path`; `event` requires `--event-name`; `outbound` requires an exact `--outbound-url`; and `scroll_depth` requires both `--page-path` and a whole `--scroll-depth-threshold` from `1` to `100`.

**Statistics**

```bash
clics goals stats <goal-id> --date-range last30days --timezone UTC
clics goals stats <goal-id> --start 2026-07-01 --end 2026-07-31 --timezone Europe/London
clics goals stats <goal-id> --domain app.example.com --ai-provider chatgpt
```

`--ai-provider` accepts `chatgpt`, `claude`, `gemini`, `perplexity`, or `copilot`.

Optional environment:

```bash
clics goals create <project-id> --goal-type page --display-name "Home" --page-path / --env-id development
```

**Update**

```bash
clics goals update <goal-id> --goal-type page --display-name "Signup" --page-path /signup
clics goals update <goal-id> --goal-type event --display-name "Purchase" --event-name purchase
clics goals update <goal-id> --goal-type outbound --display-name "Partner signup" --outbound-url https://partner.example.com/signup
clics goals update <goal-id> --goal-type scroll_depth --display-name "Read 75%" --page-path /pricing --scroll-depth-threshold 75
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

**Statistics**

```bash
clics funnels stats <funnel-id> --date-range last30days --timezone UTC
clics funnels stats <funnel-id> --start 2026-07-01 --end 2026-07-31 --timezone Europe/London
clics funnels stats <funnel-id> --domain app.example.com --ai-provider chatgpt
```

`--ai-provider` accepts `chatgpt`, `claude`, `gemini`, `perplexity`, or `copilot`.

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

### Sessions

**List**

```bash
clics sessions filter-values <project-id> --field country --date-range last30days
clics sessions filter-values <project-id> --field referrer --domain app.example.com --timezone Europe/London --limit 100
clics sessions list <project-id>
clics sessions list <project-id> --date-range last7days --limit 20
clics sessions list <project-id> --domain example.com --date-range last30days
clics sessions list <project-id> --start 2026-07-01 --end 2026-07-15
clics sessions list <project-id> --date-range last30days --timezone Europe/London
clics sessions list <project-id> --cursor "<cursor>"
```

The available session-filter fields are `country`, `device`, `browser`, `os`, `page_entry`, `page_exit`, and `referrer`.

**Get**

```bash
clics sessions get <project-id> <session-id>
clics sessions get <project-id> <session-id> --date-range last30days
clics sessions get <project-id> <session-id> --timezone Europe/London
```

**Events**

```bash
clics sessions events <project-id> <session-id>
clics sessions events <project-id> <session-id> --domain localhost
clics sessions events <project-id> <session-id> --timezone Europe/London
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
clics query <project-id> --metrics visitors pageviews --date-range last30days --timezone Europe/London
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

**Events and AI analytics**

```bash
clics query <project-id> --metrics visitors events conversion_rate --date-range last30days --dimensions event:name
clics query <project-id> --metrics visitors events conversion_rate --date-range last30days --dimensions event:outbound_url
clics query <project-id> --metrics visitors pageviews --date-range last30days --dimensions referrer:ai_provider
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
  "timezone": "Europe/London",
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

**Allowed dimensions:** `event:page` · `event:hostname` · `event:name` · `event:outbound_url` · `visit:country` · `visit:device` · `visit:browser` · `visit:os` · `visit:referrer` · `referrer:ai_provider` · `visit:utm_source` · `visit:utm_medium` · `visit:utm_campaign` · `visit:utm_term` · `visit:utm_content` · `time` · `time:hour` · `time:day`

**Additional filter-only dimension:** `event:goal`.

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

1. Run `list_goals` or `clics goals list <project-id>` (add `--env-id development` if needed), then inspect one with `get_goal` / `clics goals get <goal-id>` when its exact rule matters.
2. Run `list_funnels` or `clics funnels list <project-id>`.
3. Optionally run `get_funnel` or `clics funnels get <funnel-id>` for step details.
4. Run `get_goal_stats` / `clics goals stats <goal-id>` or `get_funnel_stats` / `clics funnels stats <funnel-id>` when the user asks for performance, rather than attempting to reconstruct it from raw rows.
5. Report what is configured. Do not create or delete resources unless the user asked.


### 4. Inspect recent sessions

1. Resolve `project_id` as above.
2. Use `list_session_filter_values` / `clics sessions filter-values <project-id> --field <field>` first when you need to discover valid session-filter values.
3. Run `list_sessions` or `clics sessions list <project-id> --date-range last7days --limit 20`.
4. Pick a `session_id`, then `get_session` / `clics sessions get <project-id> <session-id>` and `list_session_events` / `clics sessions events <project-id> <session-id>`.
5. Summarize landing/exit, bounce, duration, and notable events. Do not invent session IDs.

## Manual verification

Confirm authentication and API access with the CLI so JSON is visible in the terminal:

```bash
clics projects list
clics query <project-id> --metrics visitors --date-range last7days
clics sessions list <project-id> --date-range last7days --limit 5
```

Expect a JSON projects payload, a stats `results` array, and a sessions list. Common errors:

- `API key is required. Run clics init.` — run `clics init --api-key "…"`
- `401` / `invalid key` — rotate or recreate the key, then run `clics init` again
- `403` / `UPGRADE_REQUIRED` — a paid plan is required for API access

## Links

- Product: https://clics.dev
- Dashboard: https://platform.clics.dev
- Docs: https://docs.clics.dev
- MCP: `@clicsdev/mcp`
- CLI: `@clicsdev/cli`
