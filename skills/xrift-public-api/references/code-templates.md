# XRift Public API Code Templates

All examples use the built-in `fetch` (Node.js 18+ or modern browsers) and TypeScript. Keep the API key on a server or in a script — never in browser code that ships to users.

## Minimal Client

```typescript
const BASE_URL = 'https://api.xrift.net/api/public/v1';

export class XriftApiError extends Error {
  constructor(
    public readonly status: number,
    message: string,
    public readonly retryAfterSeconds?: number,
  ) {
    super(message);
    this.name = 'XriftApiError';
  }
}

export interface Page<T> {
  data: T[];
  pagination: { total: number; limit: number; offset: number };
}

export async function xriftGet<T>(
  path: string,
  params: Record<string, string | number | undefined> = {},
): Promise<T> {
  const apiKey = process.env.XRIFT_API_KEY;
  if (!apiKey) throw new Error('XRIFT_API_KEY is not set');

  const url = new URL(BASE_URL + path);
  for (const [key, value] of Object.entries(params)) {
    if (value !== undefined) url.searchParams.set(key, String(value));
  }

  const res = await fetch(url, {
    headers: { Authorization: `Bearer ${apiKey}` },
  });

  if (!res.ok) {
    const body = (await res.json().catch(() => ({}))) as { error?: string };
    const reset = Number(res.headers.get('X-RateLimit-Reset'));
    const retryAfterSeconds =
      res.status === 429 && reset ? Math.max(0, reset - Math.floor(Date.now() / 1000)) : undefined;
    throw new XriftApiError(res.status, body.error ?? res.statusText, retryAfterSeconds);
  }

  return (await res.json()) as T;
}
```

## Fetch One Page

```typescript
interface MarketplaceItem {
  id: string;
  name: string;
  description: string | null;
  thumbnailUrl: string | null;
  acquireCount: number;
  owner: { id: string; displayName: string; userIconUrl?: string | null };
}

// Most popular items
const popular = await xriftGet<Page<MarketplaceItem>>('/marketplace/items', {
  sort: 'popular',
  limit: 10,
});

for (const item of popular.data) {
  console.log(`${item.name} by ${item.owner.displayName} (${item.acquireCount})`);
}
```

## Walk Through Every Page

```typescript
async function fetchAll<T>(
  path: string,
  params: Record<string, string | number | undefined> = {},
): Promise<T[]> {
  const limit = 50; // the maximum
  const results: T[] = [];

  for (let offset = 0; ; offset += limit) {
    const page = await xriftGet<Page<T>>(path, { ...params, limit, offset });
    results.push(...page.data);
    if (page.data.length === 0 || results.length >= page.pagination.total) break;
  }

  return results;
}

// Every marketplace item by one creator
const items = await fetchAll<MarketplaceItem>('/marketplace/items', {
  ownerId: 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx',
  sort: 'newest',
});
```

Each page counts toward the rate limit (1,000 requests per hour by default). Cache results instead of re-fetching everything on every run.

## Upcoming Events This Week

```typescript
interface XriftEvent {
  id: string;
  seriesId?: string;
  title: string;
  status: 'upcoming' | 'live' | 'ended' | 'cancelled';
  startAt: string;
  endAt: string | null;
  worldId: string;
  repeatFrequency: string;
}

const now = new Date();
const weekLater = new Date(now.getTime() + 7 * 24 * 60 * 60 * 1000);

const events = await xriftGet<Page<XriftEvent>>('/events', {
  from: now.toISOString(),
  to: weekLater.toISOString(), // at most 366 days after `from`
  status: 'upcoming',
  limit: 50,
});

for (const event of events.data) {
  console.log(`${event.startAt} ${event.title}`);
}
```

## Event Details (Including a Single Occurrence)

Occurrence IDs of recurring events contain `:` — always encode them.

```typescript
async function getEvent(id: string): Promise<XriftEvent | null> {
  try {
    const res = await xriftGet<{ data: XriftEvent }>(`/events/${encodeURIComponent(id)}`);
    return res.data;
  } catch (error) {
    // 404: unknown event, cancelled occurrence, or a time not in the series
    if (error instanceof XriftApiError && error.status === 404) return null;
    throw error;
  }
}

// An occurrence ID from the list returns that occurrence's own time and edits
const occurrence = await getEvent(events.data[0].id);

// The series ID returns the series (first occurrence's time)
if (occurrence?.seriesId) {
  const series = await getEvent(occurrence.seriesId);
}
```

## A World and Its Live Instances

```typescript
interface World {
  id: string;
  name: string;
  isPublic: boolean;
  totalVisitCount: number;
}

interface Instance {
  id: string;
  name: string;
  currentUsers: number;
  maxCapacity: number;
}

const worldId = 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx';

// Independent requests: run them in parallel
const [world, instances] = await Promise.all([
  xriftGet<{ data: World }>(`/worlds/${worldId}`), // read:worlds
  xriftGet<Page<Instance>>(`/worlds/${worldId}/instances`), // read:instances
]);

const online = instances.data.reduce((sum, i) => sum + i.currentUsers, 0);
console.log(`${world.data.name}: ${online} online in ${instances.pagination.total} instances`);
```

## Online Friends and Where They Are

```typescript
interface OnlineFriend {
  user: { id: string; displayName: string; userIconUrl: string | null };
  instanceId: string | null; // null: location not shown (private) or not in any instance
  instanceName: string | null;
  worldId: string | null;
  worldName: string | null;
  connectedAt: string | null;
}

// read:friends. No pagination: every online friend comes back at once
const { data: friends } = await xriftGet<{ data: OnlineFriend[] }>('/friends/online');

for (const friend of friends) {
  const where = friend.instanceId
    ? `${friend.worldName ?? 'a world'} / ${friend.instanceName}`
    : 'somewhere private';
  console.log(`${friend.user.displayName}: ${where}`);
}
```

To watch friends over time, poll every few minutes at most — each call counts toward the key's rate limit.

## Error Handling

```typescript
try {
  await xriftGet('/marketplace/items', { limit: 10 });
} catch (error) {
  if (!(error instanceof XriftApiError)) throw error; // network error, etc.

  switch (error.status) {
    case 400:
      // Invalid query parameter. Messages on marketplace/event endpoints are in Japanese,
      // so branch on the status code, not the text.
      console.error('Bad request:', error.message);
      break;
    case 401:
      console.error('API key is missing, invalid, deactivated, or expired');
      break;
    case 403:
      console.error('Missing scope:', error.message); // "Insufficient scope. Required: read:xxx"
      break;
    case 404:
      console.error('Not found');
      break;
    case 429:
      console.error(`Rate limited. Retry in ${error.retryAfterSeconds ?? 60}s`);
      break;
    default:
      console.error(`Unexpected error ${error.status}:`, error.message);
  }
}
```

## curl

```bash
curl -H "Authorization: Bearer $XRIFT_API_KEY" \
  "https://api.xrift.net/api/public/v1/marketplace/items?sort=newest&limit=5"

# Occurrence ID: encode the colons
curl -H "Authorization: Bearer $XRIFT_API_KEY" \
  "https://api.xrift.net/api/public/v1/events/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx%3A%3A2026-12-02T13%3A00%3A00.000Z"
```
