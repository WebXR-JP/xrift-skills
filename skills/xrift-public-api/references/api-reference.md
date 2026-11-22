# XRift Public API v1 Reference

**Base URL**: `https://api.xrift.net/api/public/v1`

Every request needs `Authorization: Bearer <API key or CLI token>`. Lists return `{ data: [...], pagination: { total, limit, offset } }`; single resources return `{ data: {...} }`; errors return `{ error: string }`.

Common pagination parameters (where listed):

| Parameter | Type | Default | Notes |
|---|---|---|---|
| `limit` | integer | 20 | 1–50. Values above 50 are treated as 50 |
| `offset` | integer | 0 | 0 or greater |

Marketplace and event endpoints return `400` for malformed `limit` / `offset` (non-integers such as `1.5`, `1e2`, `0x10`, or `limit=0`). Other endpoints fall back to the defaults.

---

## Worlds

### `GET /worlds`

Public worlds. **Scope**: `read:worlds`. **Params**: `limit`, `offset`.

### `GET /worlds/search`

Worlds whose name, description, or creator name contains the keyword. **Scope**: `read:worlds`.

| Parameter | Required | Description |
|---|---|---|
| `q` | Yes | Keyword (`400` if missing) |
| `limit`, `offset` | - | Pagination |

### `GET /worlds/popular`

Popularity ranking. **Scope**: `read:worlds`. The response also includes `period`.

| Parameter | Default | Description |
|---|---|---|
| `period` | `all` | `all` / `week` / `month` (`400` otherwise) |
| `limit`, `offset` | - | Pagination |

### `GET /worlds/:id`

World details. **Scope**: `read:worlds`. `404 { "error": "World not found" }` if it doesn't exist, or if it is unlisted (`isPublic: false`) and you are not the owner.

World object (main fields):

| Field | Type | Description |
|---|---|---|
| `id` | string | World ID |
| `name` | string | |
| `description` | string \| null | |
| `worldType` | string | World type |
| `thumbnailUrl` | string \| null | |
| `githubUrl` | string \| null | Public repository link set by the owner |
| `status` | string | `ACTIVE` / `INACTIVE` / `PENDING` / `REJECTED` |
| `currentVersionNumber` | number | `0` if no version is published |
| `instanceCount` | number | Number of public instances |
| `totalVisitCount` / `uniqueVisitorCount` / `favoriteCount` | number | |
| `isPublic` | boolean | `false` = unlisted (reachable by URL, hidden from lists) |
| `owner` | `{ id, displayName, userIconUrl? }` \| null | |
| `createdAt` / `updatedAt` | string (ISO 8601) | |

### `GET /worlds/:id/stats`

**Scope**: `read:worlds`. Same visibility rule as `GET /worlds/:id`.

```json
{
  "data": {
    "worldId": "…",
    "totalVisits": 12345,
    "uniqueVisitors": 6789,
    "visitsLast7Days": 234,
    "visitsLast30Days": 890
  }
}
```

### `GET /worlds/:id/instances`

Public instances of a world. **Scope**: `read:instances`. **Params**: `limit`, `offset`. `404` with the same visibility rule as `GET /worlds/:id`.

---

## Users

### `GET /users/search`

Users whose display name contains the keyword (case-insensitive). **Scope**: `read:users`.

| Parameter | Required | Description |
|---|---|---|
| `q` | Yes | Keyword (`400` if missing) |
| `limit`, `offset` | - | Pagination |

Each element: `{ id, displayName, userIconUrl, createdAt }`.

### `GET /users/:id`

Public profile. **Scope**: `read:users`. `404 { "error": "User not found" }`.

Fields: `id`, `displayName`, `userIconUrl`, `bio`, `twitterUrl`, `githubUrl`, `websiteUrl`, `youtubeUrl`, `discordUsername`, `createdAt`. Profile fields are `null` when not set.

### `GET /users/:id/worlds`

A user's public worlds. **Scope**: `read:worlds`. **Params**: `limit`, `offset`. `404` if the user doesn't exist.

---

## Instances

Instances are deleted 5 minutes after everyone leaves, so stored IDs can later return `404`. Only XRift Plus permanent instances persist.

### `GET /instances`

Public instances. **Scope**: `read:instances`.

| Parameter | Description |
|---|---|
| `worldId` | Only instances of this world |
| `limit`, `offset` | Pagination |

Instance object (main fields):

| Field | Type | Description |
|---|---|---|
| `id` | string | Instance ID |
| `worldId` | string | |
| `name` | string | |
| `currentUsers` | number | People in the instance right now, guests included. Use this for the head count |
| `maxCapacity` | number | Maximum number of people the instance accepts |
| `isPublic` | boolean | |
| `allowGuests` | boolean | Whether unregistered guests can join |
| `kind` | string | `MANUAL` (created by a user) / `EVENT` (event venue) / `SOLO` |
| `eventId` | string \| null | The event this instance hosts, when `kind` is `EVENT` |
| `persistent` | boolean | XRift Plus permanent instance |
| `persistentActive` | boolean | Whether the permanent setting is in effect (within the owner's limit). `false` means the instance is deleted once empty |
| `owner` | `{ id, displayName, userIconUrl? }` | |
| `participants` | `{ id, displayName, userIconUrl }[]` | For display only (e.g. a row of icons): at most the first 5 registered users, never guests |
| `world` | World object | Included when available |
| `createdAt` | string | |

**Count people with `currentUsers`, not `participants.length`.** `participants` is capped at 5 and excludes guests, so its length is often smaller than the real head count.

### `GET /instances/:id`

Instance details. **Scope**: `read:instances`. `404 { "error": "Instance not found" }`. Private instances are not listed but can be fetched by ID.

---

## Marketplace

Only public items that passed review are returned. Validation messages are in Japanese.

### `GET /marketplace/items`

**Scope**: `read:marketplace`.

| Parameter | Default | Description |
|---|---|---|
| `sort` | `popular` | `popular` (most acquired) / `newest` / `trending` (recent momentum; items without recent acquisitions are excluded) |
| `ownerId` | - | Only this user's listings. Empty string → `400`. Unknown user → empty list |
| `limit`, `offset` | - | Pagination (strict) |

Item object:

| Field | Type | Description |
|---|---|---|
| `id` | string | Item ID |
| `name` | string | |
| `description` | string \| null | |
| `sceneUrl` | string \| null | URL of the item bundle (`remoteEntry.js`) |
| `thumbnailUrl` | string \| null | |
| `githubUrl` | string \| null | |
| `fileSize` | string | Bytes, as a string |
| `status` | string | Always `ACTIVE` here |
| `securityScore` | number \| null | |
| `isPublic` | boolean | Always `true` here |
| `currentVersionNumber` | number | |
| `acquireCount` | number | Times added to a library |
| `permissions` | `{ allowedDomains: string[], allowedCodeRules: string[] }` | |
| `owner` | `{ id, displayName, userIconUrl? }` | Creator |
| `createdAt` / `updatedAt` | string | |

Errors: `400` for invalid `sort`, empty `ownerId`, or malformed `limit` / `offset`.

### `GET /marketplace/items/search`

Partial match on name, description, or creator name. **Scope**: `read:marketplace`.

| Parameter | Required | Default | Description |
|---|---|---|---|
| `q` | Yes | - | Keyword. Missing or whitespace only → `400 { "error": "Query parameter \"q\" is required" }` |
| `sort` | - | `newest` | `popular` / `newest` / `trending` |
| `ownerId` | - | - | Only this user's listings |
| `limit`, `offset` | - | - | Pagination (strict) |

### `GET /marketplace/items/:id`

**Scope**: `read:marketplace`. `404 { "error": "Item not found" }` if it doesn't exist, is private, or is under review — even with the owner's key.

---

## Events

Validation messages are in Japanese.

### `GET /events`

**Scope**: `read:events`. Recurring events are expanded into occurrences inside `from`–`to`.

| Parameter | Required | Default | Description |
|---|---|---|---|
| `from` | Yes | - | Period start (ISO 8601) |
| `to` | Yes | - | Period end (ISO 8601). At most 366 days after `from` |
| `worldId` | - | - | Venue world |
| `instanceId` | - | - | Instance |
| `ownerId` | - | - | Organizer. Empty string → `400` |
| `kind` | - | - | `scheduled` / `instant` |
| `status` | - | - | `upcoming` / `live` / `ended` / `cancelled` |
| `order` | - | `asc` | By start time: `asc` / `desc` |
| `limit`, `offset` | - | - | Pagination (strict) |

Event object:

| Field | Type | Description |
|---|---|---|
| `id` | string | Plain ID for one-off events; `{seriesId}::{original start ISO}` for occurrences |
| `seriesId` | string? | Series ID, only on occurrences of recurring events |
| `kind` | string | `scheduled` / `instant` |
| `status` | string | Effective status at request time |
| `title` | string | |
| `description` | string \| null | |
| `thumbnailUrl` | string \| null | Shared by the whole series |
| `worldId` | string | Venue world |
| `world` | World object? | Omitted when the world has no published version |
| `currentUsers` | number | Users at the venue right now |
| `instanceId` | string \| null | Instant events only: the running instance |
| `instance` | object? | Instant events only: `{ id, name, world, currentUsers, maxCapacity }` |
| `startAt` | string | Occurrence start (reflects "this occurrence only" edits) |
| `endAt` | string \| null | `null` = end not set |
| `repeatFrequency` | string | `none` / `daily` / `weekly` / `biweekly` / `monthly` |
| `owner` | `{ id, displayName, userIconUrl? }` | Organizer |
| `createdAt` / `updatedAt` | string | |

Errors (`400`): missing or invalid `from` / `to`, `from` after `to`, period over 366 days, invalid `kind` / `status` / `order`, empty `ownerId`, malformed `limit` / `offset`.

### `GET /events/:id`

**Scope**: `read:events`. URL-encode the ID.

- Occurrence ID (`{seriesId}::{ISO}`): returns that occurrence, same shape as a `GET /events` element
- Series ID or one-off event ID: returns the event (for a series, the first occurrence's time)
- `404 { "error": "Event not found" }`: unknown event, cancelled occurrence, a time that isn't part of the series, or a `::` ID on a one-off event

---

## Friends

### `GET /friends/online`

**Scope**: `read:friends`. The API key owner's friends who are online right now. No pagination — the response is `{ "data": [...] }`, and `data` is an empty array when nobody is online.

Not included: offline friends, pending friend requests, banned users.

| Friend's state | Returned as |
|---|---|
| In a public instance of a public world | Included with location |
| In a private instance or a private world | Included with `null` location only if the friend opted in to "show me as online in private"; otherwise not included |
| Online but not in any instance | Included with `null` location |

Online friend object:

| Field | Type | Description |
|---|---|---|
| `user` | `{ id, displayName, userIconUrl }` | `userIconUrl` is `string \| null` |
| `instanceId` | string \| null | `null` when the location isn't shown |
| `instanceName` | string \| null | |
| `worldId` | string \| null | |
| `worldName` | string \| null | `null` also when the world has no published version |
| `connectedAt` | string \| null | When the friend connected (ISO 8601) |

```json
{
  "data": [
    {
      "user": { "id": "user-uuid", "displayName": "Alice", "userIconUrl": null },
      "instanceId": "instance-uuid",
      "instanceName": "Main Lobby",
      "worldId": "world-uuid",
      "worldName": "My World",
      "connectedAt": "2026-11-22T10:00:00.000Z"
    }
  ]
}
```

---

## Stats

### `GET /stats`

Platform totals. **No scope required** (any valid key).

```json
{
  "data": {
    "totalUsers": 15000,
    "totalPublicWorlds": 3200,
    "totalPublicInstances": 850
  }
}
```

---

## Authentication and Rate Limit Errors

| Status | Response |
|---|---|
| 401 | `Authentication required` / `Invalid token format` / `Invalid API key` / `API key is deactivated` / `API key has expired` / `Invalid CLI token` / `CLI token has expired` |
| 403 | `Insufficient scope. Required: read:xxx` |
| 429 | `Rate limit exceeded` |

Rate limit headers (API keys only): `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix time in seconds). The default limit is 1,000 requests per hour.
