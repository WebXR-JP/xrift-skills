---
name: xrift-public-api
description: Guide for reading XRift platform data over the Public API v1 (REST) with an API key. Covers authentication, scopes, pagination and validation rules, rate limits, and every read endpoint — worlds, users, instances, marketplace items, events, your online friends, and platform stats. Use when building bots, dashboards, or external tools that fetch XRift data.
---

# XRift Public API Guide

A guide for fetching XRift data from external tools (bots, dashboards, scripts, websites) through the Public API v1.

## References

- [API Reference](references/api-reference.md) - Every endpoint with parameters, response shapes, and errors
- [Code Templates](references/code-templates.md) - Ready-to-use `fetch` examples: a small client, pagination, recurring events, error handling

## Critical Rules

1. **Never hardcode API keys** — Read the key from an environment variable or a secret store. Never commit it, never ship it in browser code (anyone can read it there), and never paste it into logs or chat.
2. **Request only the scopes you need** — Each endpoint requires one scope. A missing scope returns `403`.
3. **Respect the rate limit** — 1,000 requests per hour per API key by default. On `429`, wait until `X-RateLimit-Reset` before retrying. Don't poll in tight loops.
4. **URL-encode event IDs** — Occurrences of recurring events have IDs like `{seriesId}::{ISO time}`, which contain `:`. Always pass IDs through `encodeURIComponent`.
5. **Send only valid integers for `limit` / `offset` on marketplace and event endpoints** — `1.5`, `1e2`, `0x10`, or `limit=0` return `400`. `limit` above 50 is treated as 50.
6. **This API is read-only for data access** — Uploading worlds and items is done with `@xrift/sdk` (see the `xrift-sdk` skill), not with these endpoints.

## Overview

**Base URL**: `https://api.xrift.net/api/public/v1`

All responses are JSON. Lists are wrapped as:

```json
{
  "data": [ ... ],
  "pagination": { "total": 99, "limit": 20, "offset": 0 }
}
```

Single resources are wrapped as `{ "data": { ... } }`. Errors are `{ "error": "message" }`.

## Authentication

Send the token in the `Authorization` header.

| Method | Header | Use case |
|---|---|---|
| API key | `Authorization: Bearer xrift_sk_xxx` | Third-party tools (scoped + rate limited) |
| CLI token | `Authorization: Bearer xrf_xxx` | XRift CLI / internal tools (all scopes, no rate limit) |

API keys are created on the [XRift settings page](https://app.xrift.net/settings). Choose scopes and an optional expiry when creating the key. The raw key is shown only once.

## Scopes

| Scope | Endpoints |
|---|---|
| `read:worlds` | `GET /worlds`, `/worlds/search`, `/worlds/popular`, `/worlds/:id`, `/worlds/:id/stats`, `/users/:id/worlds` |
| `read:users` | `GET /users/search`, `/users/:id` |
| `read:instances` | `GET /instances`, `/instances/:id`, `/worlds/:id/instances` |
| `read:marketplace` | `GET /marketplace/items`, `/marketplace/items/search`, `/marketplace/items/:id` |
| `read:events` | `GET /events`, `/events/:id` |
| `read:friends` | `GET /friends/online` |
| (none) | `GET /stats` — any valid key works |

## Endpoint Summary

| Endpoint | What it returns |
|---|---|
| `GET /worlds` | Public worlds |
| `GET /worlds/search?q=` | Worlds matching a keyword |
| `GET /worlds/popular?period=` | Popularity ranking (`all` / `week` / `month`) |
| `GET /worlds/:id` | World details |
| `GET /worlds/:id/stats` | Visit counts |
| `GET /worlds/:id/instances` | Public instances of a world |
| `GET /users/search?q=` | Users by display name |
| `GET /users/:id` | Public profile |
| `GET /users/:id/worlds` | A user's public worlds |
| `GET /instances` | Public instances (optionally `?worldId=`) |
| `GET /instances/:id` | Instance details |
| `GET /marketplace/items` | Items on the marketplace (`sort`, `ownerId`) |
| `GET /marketplace/items/search?q=` | Marketplace search |
| `GET /marketplace/items/:id` | Item details |
| `GET /events?from=&to=` | Events in a period (recurring events expanded) |
| `GET /events/:id` | Event or a single occurrence |
| `GET /friends/online` | The key owner's friends who are online, and where they are |
| `GET /stats` | Platform totals |

See [API Reference](references/api-reference.md) for full details.

## Pagination

- `limit` defaults to 20, maximum 50. `offset` defaults to 0.
- Use `pagination.total` to know when to stop. Stop when `offset + data.length >= total` or when a page comes back empty.
- Marketplace and event endpoints reject malformed `limit` / `offset` with `400`. World, user, and instance endpoints silently fall back to defaults instead — still send valid integers everywhere.

## Concepts Worth Knowing

### Marketplace

- Only public items that passed review are returned. Private items and items under review are `404` even with the owner's key.
- "Items by this creator" is `GET /marketplace/items?ownerId={userId}`. There is no `/users/:id/items`.
- An `ownerId` that doesn't exist returns an empty list, not `404`.
- The marketplace is organized by listing type (`/marketplace/items`). Other listing types may be added later under `/marketplace/...`.

### Events

- `from` and `to` (ISO 8601, at most 366 days apart) are required for `GET /events`.
- Recurring events are expanded into occurrences within the period. Each occurrence has `id = "{seriesId}::{original start ISO}"` and a `seriesId` field. One-off events have a plain `id` and no `seriesId`.
- `GET /events/{occurrenceId}` returns that occurrence (its own time, plus "this occurrence only" edits). `GET /events/{seriesId}` returns the series with the first occurrence's time. Cancelled occurrences and times not in the series return `404`.
- `status` (`upcoming` / `live` / `ended` / `cancelled`) is computed at request time.

### Friends

- `GET /friends/online` returns the **API key owner's** friends who are online right now. Offline friends, pending requests, and banned users are not included. There is no pagination (`{ "data": [...] }` only).
- Location (`instanceId`, `instanceName`, `worldId`, `worldName`) is shown only when the friend is in a public instance of a public world. A friend in a private instance or a private world is included only if they opted in to "show me as online in private", and then with the location set to `null`. A friend who is online but not in any instance is also included with `null` location. Always handle `null` location.
- Friends' whereabouts are shared with the key owner only. Don't post them anywhere public (a public channel, a website) without the friends' consent.
- To follow friends over time, poll every few minutes at most, not every few seconds — the rate limit is shared with every other request made with the key.

### Instances

- Instances are deleted automatically 5 minutes after everyone leaves, so a saved instance ID can later return `404`. Only XRift Plus permanent instances stay.

## Common Errors

| Status | Meaning |
|---|---|
| 400 | Invalid query parameter (message says which one) |
| 401 | Missing, malformed, invalid, deactivated, or expired token |
| 403 | Missing scope — `Insufficient scope. Required: read:xxx` |
| 404 | Resource doesn't exist or isn't visible |
| 429 | Rate limit exceeded |

Validation messages on marketplace and event endpoints are in Japanese (for example `limit は 1 以上の整数を指定してください`). Branch on the status code, not the message text.

## Rate Limiting

- 1,000 requests per hour per API key by default (fixed one-hour window). CLI tokens are not limited.
- Read the actual limit from `X-RateLimit-Limit` rather than assuming 1,000.
- Headers on API key requests: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix timestamp, seconds).

## Related Resources

- [XRift Documentation: Public API v1](https://xrift-docs.vercel.app/public-api/v1)
- `xrift-sdk` skill — uploading worlds and items
