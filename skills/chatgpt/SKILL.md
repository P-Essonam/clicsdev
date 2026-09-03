---
name: chatgpt
description: Use Clics through the remote MCP connector in ChatGPT. Use when a user wants ChatGPT to query Clics analytics, inspect or manage Clics projects, goals, funnels, sessions, AI crawler activity, or set up browser and AI crawler tracking from a connected Clics workspace.
---

# ChatGPT Clics Remote MCP

Use the Clics remote MCP connector in ChatGPT.

## Connection

The remote MCP endpoint is:

```txt
https://api.clics.dev/v1/mcp
```

In ChatGPT, the user connects Clics from Plugins or Connectors, signs in to
Clics, and selects a workspace.

Confirm the connected tool list before calling a tool. If the connector is not
connected, ask the user to connect Clics in ChatGPT first.

## Tool Workflow

Start with `list_projects` unless the user already gave a `project_id`.

Use project tools before analytics tools:

1. Resolve the project with `list_projects` or `get_project`.
2. If there is no project, follow Empty Workspace Setup.
3. Use goals, funnels, sessions, stats, or AI crawler tools against that project.
4. Report the project name or `project_id`, date range, filters, and environment.

## Tools

### Projects

Use projects to choose the analytics scope and to set up tracking.

| Tool | Use it for | Main input |
| --- | --- | --- |
| `list_projects` | List workspace projects. Call this first when no project is known. | none |
| `get_project` | Inspect one project. | `project_id` |
| `create_project` | Create a project for a new site. | `name`, `website_url`, optional `allow_localhost` |
| `update_project` | Rename a project or update its domain/dev setting. | `project_id`, optional `name`, `website_url`, `allow_localhost` |
| `delete_project` | Delete a project. Confirm intent first because analytics data is tied to it. | `project_id` |

Example:

```json
{
  "name": "Acme",
  "website_url": "https://acme.com",
  "allow_localhost": true
}
```

### Goals

Use goals to track conversions.

| Tool | Use it for | Main input |
| --- | --- | --- |
| `list_goals` | List goals for a project. | `project_id` |
| `create_goal` | Create a conversion goal. | `project_id`, `env_id`, `goal_type`, `display_name`, `rule` |
| `update_goal` | Update a goal. | `goal_id`, optional `display_name`, `goal_type`, `rule` |
| `delete_goal` | Delete a goal. | `goal_id` |

Goal types and rules:

| Type | Rule |
| --- | --- |
| `page` | `{ "page_path": "/signup" }` |
| `event` | `{ "event_name": "purchase" }` |
| `outbound` | `{ "outbound_url": "https://partner.example.com/signup" }` |
| `scroll_depth` | `{ "page_path": "/pricing", "scroll_depth_threshold": 75 }` |

### Funnels

Use funnels to measure ordered conversion paths.

| Tool | Use it for | Main input |
| --- | --- | --- |
| `list_funnels` | List funnels for a project. | `project_id` |
| `get_funnel` | Inspect one funnel definition. | `funnel_id` |
| `create_funnel` | Create a funnel with ordered steps. | `project_id`, `name`, `steps`, optional conversion window and filters |
| `update_funnel` | Update a funnel definition. | `funnel_id`, optional `name`, `conversion_window`, `steps` |
| `delete_funnel` | Delete a funnel. | `funnel_id` |

Funnel conversion window units:

```txt
seconds, minutes, hours, days, weeks, months
```

Funnel filter fields:

```txt
country, device, browser, os, page, event, hostname, referrer,
utm_source, utm_medium, utm_campaign, utm_term, utm_content
```

Funnel filter operators:

```txt
is, is_not, contains, not_contains
```

### Sessions

Use sessions to inspect individual visits and their events.

| Tool | Use it for | Main input |
| --- | --- | --- |
| `list_sessions` | List recent sessions with optional domain, date range, and pagination. | `project_id`, optional `domain`, `date_range`, `start`, `end`, `cursor`, `limit` |
| `get_session` | Inspect one session. | `project_id`, `session_id` |
| `list_session_events` | List events in one session. | `project_id`, `session_id` |

### Stats

Use `query_stats` for dashboard-style analytics.

Main input:

- `project_id`
- `date_range`
- optional `metrics`
- optional `dimensions`
- optional `filters`
- optional `domain`
- optional `timezone`

Metrics:

```txt
visitors, visits, pageviews, bounce_rate, visit_duration,
views_per_visit, conversion_rate, events
```

Dimensions:

```txt
event:page, event:hostname, event:name, event:outbound_url,
visit:country, visit:device, visit:browser, visit:os, visit:referrer,
referrer:ai_provider,
visit:utm_source, visit:utm_medium, visit:utm_campaign, visit:utm_term,
visit:utm_content,
time, time:hour, time:day
```

Filter-only dimension:

```txt
event:goal
```

Stats filter operators:

```txt
is, is_not, contains, contains_not
```

Example:

```json
{
  "project_id": "project_123",
  "date_range": "last7days",
  "metrics": ["visitors", "pageviews"],
  "dimensions": ["time:day"],
  "timezone": "Europe/Paris"
}
```

### AI Crawler Analytics

Use `get_ai_crawler_analytics` to inspect AI crawler activity. AI crawler
analytics are production-only; use no environment argument.

Main input:

- `project_id`
- optional `date_range`
- optional `timezone`
- optional filters: `category`, `provider`, `crawler`, `status`
- optional operators: `provider_op`, `crawler_op`, `status_op`

AI crawler categories:

```txt
answer_fetch, search_index, training
```

AI crawler filter operators:

```txt
is, is_not
```

AI crawler providers:

```txt
OpenAI, Anthropic, Perplexity, Google, Microsoft, Mistral, Amazon,
DuckDuckGo, Apple, Moonshot AI, Common Crawl
```

AI crawlers:

```txt
ChatGPT-User, OAI-SearchBot, GPTBot,
Claude-User, Claude-SearchBot, ClaudeBot,
Perplexity-User, PerplexityBot,
Google-Agent, Google-GeminiNotebook, Google-NotebookLM,
Google-Read-Aloud, Google-InspectionTool, Googlebot, GoogleOther,
Google-CloudVertexBot,
Bingbot, msnbot,
MistralAI-User, MistralAI-Index,
Amzn-User, Amzn-SearchBot, Amazonbot,
DuckAssistBot,
Applebot,
Kimi-User, Kimi-SearchBot, KimiBot,
CCBot
```

Example:

```json
{
  "project_id": "project_123",
  "date_range": "last30days",
  "timezone": "Europe/Paris",
  "provider": "OpenAI",
  "provider_op": "is",
  "category": "answer_fetch"
}
```

## Input Shapes

These examples mirror the remote MCP tool schemas. Use the live tool schema as
the final source of truth if ChatGPT shows a field-level schema.

### Project Inputs

`list_projects` accepts optional pagination:

```json
{
  "limit": 20,
  "cursor": "next_cursor"
}
```

`get_project` and `delete_project`:

```json
{
  "project_id": "project_123"
}
```

`create_project`:

```json
{
  "name": "Acme Marketing",
  "website_url": "example.com",
  "allow_localhost": false
}
```

`update_project` requires `project_id` and at least one field to change:

```json
{
  "project_id": "project_123",
  "name": "Acme",
  "website_url": "acme.com",
  "allow_localhost": true
}
```

### Goal Inputs

`list_goals`:

```json
{
  "project_id": "project_123",
  "env_id": "production"
}
```

`create_goal` uses `display_name` and `goal_type`:

```json
{
  "project_id": "project_123",
  "env_id": "production",
  "display_name": "Signup",
  "goal_type": "page",
  "rule": {
    "page_path": "/signup"
  }
}
```

`update_goal` does not take `project_id`. Pass `goal_id` and at least one field:

```json
{
  "goal_id": "goal_123",
  "display_name": "Trial signup",
  "rule": {
    "page_path": "/thank-you"
  }
}
```

`delete_goal`:

```json
{
  "goal_id": "goal_123"
}
```

### Funnel Inputs

`list_funnels` accepts optional pagination:

```json
{
  "project_id": "project_123",
  "env_id": "production",
  "limit": 20,
  "cursor": "next_cursor"
}
```

`get_funnel` and `delete_funnel` use only `funnel_id`:

```json
{
  "funnel_id": "funnel_123"
}
```

`create_funnel` requires at least two ordered steps. Each step needs a name and
at least one filter:

```json
{
  "project_id": "project_123",
  "env_id": "production",
  "name": "Checkout",
  "conversion_window": {
    "value": 7,
    "unit": "days"
  },
  "steps": [
    {
      "name": "Cart",
      "filters": [
        {
          "filter_type": "page",
          "operator": "is",
          "values": ["/cart"]
        }
      ]
    },
    {
      "name": "Purchase",
      "filters": [
        {
          "filter_type": "page",
          "operator": "is",
          "values": ["/thanks"]
        }
      ]
    }
  ]
}
```

`update_funnel` does not take `project_id`. Pass `funnel_id` and at least one
field:

```json
{
  "funnel_id": "funnel_123",
  "name": "Updated checkout"
}
```

### Session Inputs

`list_sessions`:

```json
{
  "project_id": "project_123",
  "domain": "example.com",
  "date_range": "last7days",
  "limit": 20,
  "cursor": "next_cursor"
}
```

For a custom session range, pass both `start` and `end`; when both are set,
`date_range` is ignored:

```json
{
  "project_id": "project_123",
  "start": "2026-09-01",
  "end": "2026-09-03"
}
```

`get_session` and `list_session_events`:

```json
{
  "project_id": "project_123",
  "session_id": "session_123",
  "date_range": "allTime"
}
```

### Stats Inputs

`query_stats` takes the full stats request body. `date_range` is either a
preset string or a two-item `[start, end]` ISO8601 array.

KPI query:

```json
{
  "project_id": "project_123",
  "domain": "example.com",
  "timezone": "Europe/Paris",
  "metrics": ["visitors", "pageviews", "bounce_rate"],
  "date_range": "last30days",
  "include": {
    "previous_period": true
  }
}
```

Breakdown query with filters, sort, and pagination:

```json
{
  "project_id": "project_123",
  "metrics": ["visitors", "pageviews"],
  "date_range": ["2026-09-01T00:00:00Z", "2026-09-03T23:59:59Z"],
  "dimensions": ["visit:country"],
  "filters": [["is", "visit:device", ["desktop"]]],
  "order_by": [["visitors", "desc"]],
  "pagination": {
    "limit": 50,
    "offset": 0
  }
}
```

### AI Crawler Inputs

`get_ai_crawler_analytics` uses production implicitly. `provider` and `crawler`
may be a single enum value or a non-empty array of enum values. Use
`provider_op`, `crawler_op`, and `status_op` for filter operators.

```json
{
  "project_id": "project_123",
  "date_range": "last30days",
  "timezone": "Europe/Paris",
  "category": "answer_fetch",
  "provider": ["OpenAI", "Anthropic"],
  "provider_op": "is",
  "crawler": "GPTBot",
  "crawler_op": "is_not",
  "status": "200",
  "status_op": "is",
  "breakdown_limit": 50,
  "filter_values_limit": 200
}
```

For a custom AI crawler range, pass `start` and `end`:

```json
{
  "project_id": "project_123",
  "start": "2026-09-01T00:00:00Z",
  "end": "2026-09-03T23:59:59Z"
}
```

## Shared Presets

Environment values:

```txt
production, development
```

`env_id` is used by goals and funnels. Default to `production` when the user
does not specify an environment. `query_stats` does not take `env_id`; use its
`domain` field, or pass `localhost` as the domain for development traffic.

Date range presets:

```txt
last24h, last7days, last30days, last3months, last12months,
monthToDate, quarterToDate, yearToDate, allTime
```

For a custom range, pass the explicit start and end dates in the format expected
by the tool schema.

Timezone values are IANA timezones, for example:

```txt
UTC, Europe/Paris, Europe/London, America/New_York, America/Los_Angeles
```

## Empty Workspace Setup

Start by calling `list_projects` when the user asks to inspect analytics and no
project is known. If the workspace has no projects, pause analytics work and
ask the user for:

- project name
- production website domain
- whether localhost tracking should be allowed for development

Create the project with `create_project`. After creation, use the returned
`project_id` in the setup instructions.

## Browser Tracking Setup

After creating the project, help the user install browser tracking. Read:

```txt
https://docs.clics.dev/installation-guides
```

Choose the section that matches the user's framework. If the framework or entry
file is unclear, ask where the app shell, root layout, or HTML head lives.

Give the user the browser tracking snippet with the real `project_id` already
filled in:

```html
<script
  defer
  data-project-id="<project_id>"
  src="https://clics.dev/tracker.js"
></script>
```

For Next.js, use the same `project_id` in `next/script`:

```tsx
<Script
  strategy="afterInteractive"
  src="https://clics.dev/tracker.js"
  data-project-id="<project_id>"
/>
```

Help the user place the snippet in the correct file, then tell them to open the
site and check the Clics dashboard for the first pageview.

## AI Crawler Tracking Setup

AI crawler analytics uses a separate server-side setup. Read:

```txt
https://docs.clics.dev/installation-guides/ai-crawler-tracking
```

Identify the section for the user's runtime, such as Next.js, TanStack Start,
Hono, Express, Cloudflare, or a generic Fetch handler. Help the user add the
server-side tracking code and the required project-scoped crawler token.

Use `get_ai_crawler_analytics` after setup to verify crawler activity when data
is available. Report that AI crawler analytics are production-only.

## Reporting

When reporting analytics, state:

- project used
- date range
- timezone when provided
- filters applied
- environment when the tool accepts `env_id`
- domain when the tool accepts `domain`
- for AI crawler analytics, production is implied

When a result is empty, say whether the workspace has no project, the project
has no data for that date range, or the filters removed all rows.
