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
| `create_goal` | Create a conversion goal. | `project_id`, `name`, `type`, `rule` |
| `update_goal` | Update a goal. | `project_id`, `goal_id`, optional fields |
| `delete_goal` | Delete a goal. | `project_id`, `goal_id` |

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
| `get_funnel` | Inspect one funnel definition. | `project_id`, `funnel_id` |
| `create_funnel` | Create a funnel with ordered steps. | `project_id`, `name`, `steps`, optional conversion window and filters |
| `update_funnel` | Update a funnel definition. | `project_id`, `funnel_id`, optional fields |
| `delete_funnel` | Delete a funnel. | `project_id`, `funnel_id` |

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
| `list_sessions` | List recent sessions with optional filters. | `project_id`, optional pagination, filters, date range |
| `get_session` | Inspect one session. | `project_id`, `session_id` |
| `list_session_events` | List events in one session. | `project_id`, `session_id` |

Session filter fields:

```txt
country, device, browser, os, page_entry, page_exit, referrer
```

Session filter operators:

```txt
is, is_not
```

### Stats

Use `query_stats` for dashboard-style analytics.

Main input:

- `project_id`
- `env_id`, usually `production`
- `date_range`
- optional `metrics`
- optional `dimensions`
- optional `filters`
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
  "env_id": "production",
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
- optional operators for filters

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
  "provider_operator": "is",
  "category": "answer_fetch",
  "category_operator": "is"
}
```

## Shared Presets

Environment values:

```txt
production, development
```

Default to `production` when the user does not specify an environment.

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
- environment, except for AI crawler analytics where production is implied

When a result is empty, say whether the workspace has no project, the project
has no data for that date range, or the filters removed all rows.
