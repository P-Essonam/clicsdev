---
name: clics
description: Analyze website traffic, acquisition, pages, conversions, funnels, and visitor journeys with the connected Clics MCP server. Use when the user mentions Clics or asks about their website analytics.
---

# Clics

Clics is privacy-friendly, cookieless web analytics. Use the connected `clics` MCP tools directly.

## Connection

- The user signs in through the Clics connection.
- Never ask for an API key, access token, or password.
- If the server reports that authentication is required, ask the user to connect or reconnect Clics in Cursor.
- Only use tools currently exposed by the connected server. Do not invent direct API requests.

## Working safely

- Resolve the project with `list_projects` before running analytics when no `project_id` is known.
- Use read tools without extra confirmation when they match the request.
- Create or update resources only when the user requests the change.
- Confirm the exact target before deleting a project, goal, or funnel unless the user already named it unambiguously.
- Never invent project, goal, funnel, or session identifiers.

## Tools

### Projects

| Tool             | Main inputs                                       |
| ---------------- | ------------------------------------------------- |
| `list_projects`  | optional `cursor`, `limit`                        |
| `get_project`    | `project_id`                                      |
| `create_project` | `name`, `website_url`, optional `allow_localhost` |
| `update_project` | `project_id`, optional changed fields             |
| `delete_project` | `project_id`                                      |

### Goals

`env_id` is `production` or `development`; use `production` unless the user specifies otherwise.

| Tool          | Main inputs                                                          |
| ------------- | -------------------------------------------------------------------- |
| `list_goals`  | `project_id`, optional `env_id`                                      |
| `create_goal` | `project_id`, `goal_type`, `display_name`, `rule`, optional `env_id` |
| `update_goal` | `goal_id`, changed fields                                            |
| `delete_goal` | `goal_id`                                                            |

Goal rules depend on `goal_type`:

| Goal type      | Rule                                                        |
| -------------- | ----------------------------------------------------------- |
| `page`         | `{ "page_path": "/signup" }`                                |
| `event`        | `{ "event_name": "purchase" }`                              |
| `outbound`     | `{ "outbound_url": "https://example.com/signup" }`          |
| `scroll_depth` | `{ "page_path": "/pricing", "scroll_depth_threshold": 75 }` |

Use an exact HTTP(S) URL for outbound goals. Scroll depth must be an integer from 1 to 100. When changing a goal type, send the complete rule for the new type.

### Funnels

| Tool            | Main inputs                                                                                |
| --------------- | ------------------------------------------------------------------------------------------ |
| `list_funnels`  | `project_id`, optional `env_id`, `cursor`, `limit`                                         |
| `get_funnel`    | `funnel_id`                                                                                |
| `create_funnel` | `project_id` and a body containing `name`, `conversion_window`, `steps`, optional `env_id` |
| `update_funnel` | `funnel_id` and updated body without `env_id`                                              |
| `delete_funnel` | `funnel_id`                                                                                |

### Sessions

| Tool                  | Main inputs                                                            |
| --------------------- | ---------------------------------------------------------------------- |
| `list_sessions`       | `project_id`, optional `domain`, period, `timezone`, `cursor`, `limit` |
| `get_session`         | `project_id`, `session_id`, optional scope                             |
| `list_session_events` | `project_id`, `session_id`, optional scope                             |

Use a preset such as `last24h`, `last7days`, `last30days`, or `allTime`. For custom periods, provide both `start` and `end`. Use an IANA timezone when calendar boundaries matter; otherwise use UTC.

### Analytics

Use `query_stats` with:

- `project_id`
- `metrics`
- `date_range`
- optional `domain`, `timezone`, `dimensions`, `filters`, `order_by`, `include`, and `pagination`

Example overview:

```json
{
  "project_id": "project_id",
  "metrics": ["visitors", "visits", "pageviews", "bounce_rate"],
  "date_range": "last30days",
  "timezone": "UTC",
  "include": { "previous_period": true }
}
```

Useful dimensions include `event:page`, `event:hostname`, `event:name`, `event:outbound_url`, `visit:country`, `visit:device`, `visit:browser`, `visit:os`, `visit:referrer`, `referrer:ai_provider`, UTM dimensions, and time dimensions.

Use one time dimension or at most two breakdown dimensions. For ranked results, provide `order_by` and a reasonable pagination limit.

## Common playbooks

### Website overview

1. Resolve the project.
2. Query visitors, visits, pageviews, bounce rate, visit duration, and views per visit.
3. Include the previous period when comparison is useful.
4. Summarize the largest changes and explain what they may indicate without presenting guesses as facts.

### Top pages and acquisition

1. Query top pages with `event:page`.
2. Query referrers, AI providers, or UTM dimensions depending on the request.
3. Rank by visitors or pageviews and report the date range and timezone.

### Goals and funnels

1. List the configured goals and funnels.
2. Use `get_funnel` when step details are needed.
3. Report the existing configuration before suggesting changes.
4. Do not create, update, or delete anything unless requested.

### Visitor journeys

1. List recent sessions for the requested project and period.
2. Select a real `session_id` from the results.
3. Fetch the session and its events.
4. Summarize landing page, exit page, duration, bounce behavior, and notable events.

## Errors

- Authentication required: ask the user to connect or reconnect Clics.
- Upgrade required: explain that the connected workspace does not currently have access; do not ask for credentials.
- Invalid identifier: resolve the resource again with the appropriate list tool.
- Empty data: confirm the project, environment, domain, period, and timezone before concluding that no activity exists.

## Links

- Product: https://clics.dev
- Dashboard: https://platform.clics.dev
- Documentation: https://docs.clics.dev
