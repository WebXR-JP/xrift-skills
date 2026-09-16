# @xrift/world-components API Reference

## Hooks

### useXRift()

Hook for getting asset URLs. Required for loading assets within a world.

**Returns**: `{ baseUrl: string }`

```typescript
import { useXRift } from '@xrift/world-components'

const { baseUrl } = useXRift()
// baseUrl includes a trailing /
// Correct: `${baseUrl}model.glb`
// Wrong:   `${baseUrl}/model.glb`
```

### useInstanceState(key, initialValue)

Hook for synchronizing state across all users. Shared among all players within a world instance.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `key` | `string` | Unique key for the state |
| `initialValue` | `T` | Initial value |

**Returns**: `[T, (value: T | ((prev: T) => T)) => void]`

```typescript
import { useInstanceState } from '@xrift/world-components'

const [count, setCount] = useInstanceState('click-count', 0)
setCount(prev => prev + 1)
```

### useInstanceEvent(eventName, callback)

Hook for sending and receiving instance events. Supports platform events (`user-joined`, `user-left`) for receiving, and custom events for both sending and receiving.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `eventName` | `string` | Event name |
| `callback` | `(data: T) => void` | Callback when event is received |

**Returns**: `(data: T) => void` — Emit function. Returns a no-op for platform events (`user-joined`, `user-left`).

**Event Types**:
| Type | Event Name | Send | Receive | Description |
|------|-----------|:----:|:-------:|-------------|
| Platform | `user-joined` | - | Yes | User joined the instance |
| Platform | `user-left` | - | Yes | User left the instance |
| Custom | Any string | Yes | Yes | World-specific custom events |

```typescript
import { useInstanceEvent } from '@xrift/world-components'

// Receive platform events (receive only, cannot emit)
useInstanceEvent('user-joined', (data) => {
  console.log('User joined:', data)
})

// Send and receive custom events
const emitReaction = useInstanceEvent('reaction', (data) => {
  console.log('Reaction received:', data)
})
emitReaction({ emoji: '👍', userId: 'user-1' })
```

### useUsers()

Hook for getting user information and positions.

**Returns**:
| Property | Type | Description |
|----------|------|-------------|
| `localUser` | `User` | Current user's information |
| `remoteUsers` | `User[]` | List of other users |
| `getMovement(socketId)` | `(id: string) => PlayerMovement \| null` | Get movement data for a specific user |
| `getLocalMovement()` | `() => PlayerMovement` | Get current user's movement data |
| `getAvatarHeight?(userId)` | `(id: string) => AvatarHeight \| undefined` | Get avatar height data for a specific user |
| `getLocalAvatarHeight?()` | `() => AvatarHeight` | Get current user's avatar height data |

```typescript
import { useUsers } from '@xrift/world-components'

const { localUser, remoteUsers, getMovement, getLocalMovement, getAvatarHeight, getLocalAvatarHeight } = useUsers()
```

> `getAvatarHeight` / `getLocalAvatarHeight` are optional. Use optional chaining (`?.`) when calling. Default: `{ height: 1.5, eyeHeight: 1.35 }`

### useSpawnPoint()

Hook for getting the spawn point.

**Returns**: `{ position: [number, number, number], yaw: number }`

```typescript
import { useSpawnPoint } from '@xrift/world-components'

const { position, yaw } = useSpawnPoint()
```

### useScreenShareContext()

Hook for screen sharing state.

**Returns**:
| Property | Type | Description |
|----------|------|-------------|
| `videoElement` | `HTMLVideoElement \| null` | Screen share video element |
| `isSharing` | `boolean` | Whether currently sharing |
| `startScreenShare` | `() => void` | Start screen sharing |
| `stopScreenShare` | `() => void` | Stop screen sharing |

```typescript
import { useScreenShareContext } from '@xrift/world-components'

const { videoElement, isSharing, startScreenShare, stopScreenShare } = useScreenShareContext()
```

### useConfirm()

Hook for showing a confirmation modal to the user. Useful for confirming important actions like world navigation. Also serves as a workaround for iOS Safari's popup blocker — by prompting user interaction through the confirmation dialog, it creates a user-gesture event chain that allows `window.open` and external navigation to proceed without being blocked.

**Returns**: `{ requestConfirm: (options: ConfirmOptions) => Promise<boolean> }`

`ConfirmOptions`:
| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `title` | `string` | No | Dialog title |
| `message` | `string` | Yes | Message displayed to the user |
| `confirmLabel` | `string` | No | Label for the confirm button |
| `cancelLabel` | `string` | No | Label for the cancel button |

The returned `Promise<boolean>` resolves to `true` if the user confirms, `false` if cancelled.

```typescript
import { useConfirm } from '@xrift/world-components'

const { requestConfirm } = useConfirm()
const ok = await requestConfirm({ message: 'ワールドを移動しますか？' })
if (ok) {
  // proceed with action
}
```

### useTeleport()

Hook for instant player teleportation.

**Returns**: `{ teleport: (dest: TeleportDestination) => void }`

`TeleportDestination`: `{ position: [number, number, number], yaw?: number }`
- `yaw` is in degrees (0-360). If omitted, the player's current facing direction is preserved.

```typescript
import { useTeleport } from '@xrift/world-components'

const { teleport } = useTeleport()
teleport({ position: [50, 0, 30], yaw: 180 })
```

[useTeleport Documentation](https://docs.xrift.net/world-components/components/#useteleport)

### useBillboardY()

Hook that returns a ref which automatically rotates the target Object3D to face the camera on the Y-axis only each render pass. Uses a sentinel Mesh's `onBeforeRender` internally, so it works correctly with Mirror (Reflector) — the virtual camera is used for rotation calculation during mirror render passes.

**Returns**: `RefObject<T>` — Ref to attach to the target Object3D.

```typescript
import { useBillboardY } from '@xrift/world-components'
import type { Mesh } from 'three'

const ref = useBillboardY<Mesh>()
<mesh ref={ref}>...</mesh>
```

### useInstance(instanceId)

Hook for fetching instance information and navigating to an instance with a confirmation dialog. Internally uses `InstanceContext` (injected by the platform) and `useConfirm` for the confirmation modal.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `instanceId` | `string` | Target instance ID |

**Returns**:
| Property | Type | Description |
|----------|------|-------------|
| `info` | `InstanceInfo \| null` | Instance information (fetched on mount) |
| `navigateWithConfirm` | `() => Promise<void>` | Navigate to the instance with a confirmation modal |

`navigateWithConfirm` fetches the latest instance info, shows a confirmation dialog with the world name, instance name, and current user count, then navigates on confirmation.

```typescript
import { useInstance } from '@xrift/world-components'

const { info, navigateWithConfirm } = useInstance('target-instance-id')
// info?.world.name — world name
// info?.currentUsers — current user count
// navigateWithConfirm() — navigate with confirmation
```

### useWorld(worldId)

Hook for fetching world information. Internally uses `WorldContext` (injected by the platform).

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `worldId` | `string` | Target world ID |

**Returns**:
| Property | Type | Description |
|----------|------|-------------|
| `info` | `WorldInfo \| null` | World information (fetched on mount) |

```typescript
import { useWorld } from '@xrift/world-components'

const { info } = useWorld('target-world-id')
// info?.name — world name
// info?.thumbnailUrl — thumbnail URL
```

### useVoiceVolumeOverride()

Hook for overriding specific user's voice chat volume. Used for stages or podiums.

> Renamed from `useAudioVolume` in v0.34.0. Old name still works but is deprecated.

**Returns**:
| Property | Type | Description |
|----------|------|-------------|
| `setOverride` | `(userId: string, volume: number) => void` | Set volume override |
| `clearOverride` | `(userId: string) => void` | Clear override |
| `clearAll` | `() => void` | Clear all overrides |
| `getOverrides` | `() => ReadonlyMap<string, number>` | Get current overrides |

```typescript
import { useVoiceVolumeOverride } from '@xrift/world-components'

const { setOverride, clearOverride } = useVoiceVolumeOverride()
setOverride(speakerUserId, 1.0)
```

### useDefaultFont(locales)

Hook for loading multilingual MSDF fonts for UIKit (`@pmndrs/uikit`). Fonts are also registered globally via `setGlobalProperties` on load, so `fontFamily="ja"` works without explicitly passing `fontFamilies` to `Container`.

**Parameters**:
| Parameter | Type | Description |
|-----------|------|-------------|
| `locales` | `FontLocale[]` | Array of font locales to load |

**Returns**: `FontFamilies | undefined` — Returns `FontFamilies` once loaded, `undefined` while loading.

`FontLocale`: `'ja'`

```typescript
import { useDefaultFont } from '@xrift/world-components'
import type { FontLocale } from '@xrift/world-components'

const FONT_LOCALES: FontLocale[] = ['ja']
const fontFamilies = useDefaultFont(FONT_LOCALES)

<Container fontFamilies={fontFamilies}>
  <Text fontFamily="ja">こんにちは</Text>
</Container>
```

### useFileInput()

Hook for opening a file picker dialog from the 3D world. Displays an overlay UI with drag & drop support. If called during a VR session, the session is automatically ended before the file picker appears.

**Returns**: `{ requestFileInput: (request: FileInputRequest) => void }`

`FileInputRequest`:
| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier for the input |
| `accept` | `string` | No | Accepted file types (e.g. `'.vrm'`, `'image/*'`) |
| `multiple` | `boolean` | No | Allow multiple file selection |
| `maxSize` | `number` | No | Maximum file size in bytes |
| `onSelect` | `(files: File[]) => void` | Yes | Callback when files are selected |
| `onCancel` | `() => void` | No | Callback when cancelled |
| `onError` | `(error: FileInputError) => void` | No | Callback on error (e.g. file too large) |

`FileInputError`: `{ type: 'file_too_large' | 'invalid_type', message: string }`

```typescript
import { useFileInput } from '@xrift/world-components'

const { requestFileInput } = useFileInput()
requestFileInput({
  id: 'avatar-upload',
  accept: '.vrm',
  maxSize: 30 * 1024 * 1024,
  onSelect: (files) => { /* handle files */ },
  onError: (error) => { /* handle error */ },
})
```

---

### useSharedFile()

Hook for uploading, listing, locking (deletion protection), updating, and deleting shared files within an instance. Upload images or documents from the 3D world and share them with other users. Progress tracking is supported via an optional callback.

**Returns**: `{ uploadSharedFile, getSharedFiles, setSharedFileLock, updateSharedFile, deleteSharedFile }`

| Property | Type | Description |
|----------|------|-------------|
| `uploadSharedFile` | `(file: File, onProgress?: (progress: number) => void, options?: UploadSharedFileOptions) => Promise<SharedFileInfo>` | Upload a file with optional progress callback and optional description / metadata |
| `getSharedFiles` | `() => Promise<SharedFileInfo[]>` | Get the list of shared files |
| `setSharedFileLock` | `(fileId: string, locked: boolean) => Promise<SharedFileInfo>` | Set the lock state (deletion protection) of a file |
| `updateSharedFile` | `(fileId: string, updates: UpdateSharedFileParams) => Promise<SharedFileInfo>` | Update file info (fileName / description / metadata). Pass `null` to clear description / metadata |
| `deleteSharedFile` | `(fileId: string) => Promise<void>` | Delete a file |

`SharedFileInfo`:
| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique file ID |
| `fileName` | `string` | File name |
| `contentType` | `string` | MIME type |
| `fileSize` | `number` | File size in bytes |
| `publicUrl` | `string` | Public URL |
| `locked` | `boolean` | Whether the file is locked (deletion protection) |
| `description` | `string \| null` | Description text (up to 500 characters) |
| `metadata` | `Record<string, string> \| null` | Flat key-value metadata (up to 20 entries, keys 1-64 chars, values up to 500 chars) |
| `createdAt` | `string` | Creation date (ISO 8601) |

**Note**: A locked file rejects both `deleteSharedFile` and `updateSharedFile`. Unlock first with `setSharedFileLock(fileId, false)`, then delete / update.

```typescript
import { useSharedFile, useFileInput } from '@xrift/world-components'

const { uploadSharedFile, getSharedFiles, setSharedFileLock, updateSharedFile, deleteSharedFile } = useSharedFile()
const { requestFileInput } = useFileInput()

// Upload a file with progress tracking, description and metadata
requestFileInput({
  id: 'shared-upload',
  accept: 'image/*',
  maxSize: 10 * 1024 * 1024,
  onSelect: async (files) => {
    const result = await uploadSharedFile(
      files[0],
      (progress) => console.log(`${progress}%`),
      { description: 'Exhibit A', metadata: { exhibit: 'pedestal-1' } },
    )
    console.log('URL:', result.publicUrl)
    // Lock right after upload to prevent accidental deletion
    await setSharedFileLock(result.id, true)
  },
})

// List shared files
const files = await getSharedFiles()

// Update description / metadata (null clears the field)
await updateSharedFile(fileId, { description: 'Exhibit B', metadata: null })

// Delete a file (unlock first if locked)
await setSharedFileLock(fileId, false)
await deleteSharedFile(fileId)
```

---

### useItem()

Hook for getting information about a placed item. Each placement gets a different ID, even for the same item type.

**Returns**: `{ id: string, placedBy: ItemPlacer | null }`

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique placement ID (UUID) |
| `placedBy` | `ItemPlacer \| null` | Who placed this item. `null` when the placer cannot be identified |

**ItemPlacer**:
| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | The placer's userId. Always present (resolved by the server) |
| `displayName` | `string \| null` | Display name. `null` if the profile cannot be resolved (e.g. they left the instance) |
| `avatarUrl` | `string \| null` | Icon URL. `null` if it cannot be resolved |
| `isLocalUser` | `boolean` | Whether the local user placed it |

```typescript
import { useItem, useInstanceState } from '@xrift/world-components'

const { id, placedBy } = useItem()

// Use id to scope per-placement state
const [votes, setVotes] = useInstanceState(`votes-${id}`, 0)

// Only the person who placed it can reset the votes
if (placedBy?.isLocalUser) {
  // show a reset button
}
```

> `displayName` and `avatarUrl` are resolved on demand, so they become `null` once the placer
> leaves the instance. `id` always stays, so use it when you need a stable key.

> While the item is still a placement preview (before it is confirmed), `placedBy` is the local user.

> Requires `@xrift/world-components` >= 0.49.0

> Must be used within ItemProvider (automatically provided by the platform).

---

### useWorldStorage()

Hook for world-scoped persistent key-value storage (World Storage). Persist rankings, in-world currency, registration data, and more across instances. Unlike `useInstanceState` (volatile, per-instance), World Storage survives instance shutdowns and is shared across all instances of the world.

**Returns**: `{ shared, player }`

| Property | Type | Description |
|----------|------|-------------|
| `shared` | `SharedWorldStorage` | Shared KV (one shared value per world). Any authenticated user in the instance can write |
| `player` | `PlayerWorldStorage` | Per-player KV. Writes are limited to your own values; reads can access other users' values |

`SharedWorldStorage`:
| Method | Type | Description |
|--------|------|-------------|
| `get` | `(key: string) => Promise<unknown>` | Get a value (`undefined` if missing) |
| `list` | `() => Promise<WorldStorageEntry[]>` | Get all keys and values |
| `set` | `(key: string, value: unknown) => Promise<void>` | Save a value |
| `increment` | `(key: string, delta: number) => Promise<number>` | Atomically add to a numeric value and return the result |
| `delete` | `(key: string) => Promise<void>` | Delete a value (idempotent) |

`PlayerWorldStorage`:
| Method | Type | Description |
|--------|------|-------------|
| `get` | `(key: string, options?: { userId?: string }) => Promise<unknown>` | Get a value. Pass `userId` to read another user's value |
| `list` | `(options?: { userId?: string }) => Promise<WorldStorageEntry[]>` | Get all keys and values. Pass `userId` to read another user's values |
| `set` | `(key: string, value: unknown) => Promise<void>` | Save your own value |
| `increment` | `(key: string, delta: number) => Promise<number>` | Atomically add to your own numeric value and return the result |
| `delete` | `(key: string) => Promise<void>` | Delete your own value (idempotent) |

**Constraints**:
- Save at game-event milestones. For per-frame sync, use `useInstanceState` instead
- Capacity: 10MB total per world / 100KB per entry / 256 shared keys / 64 keys per user
- Key format: `/^[A-Za-z0-9_.\-:]{1,128}$/`
- Rate limit: 30 writes per minute per user
- Reads are public (accessible via API without authentication) — never store secrets
- Guests are read-only (writes throw `WorldStorageError` with code `UNAUTHORIZED`)
- For currency / scores, use `increment` instead of `set` (no lost updates under concurrency)

**Errors**: failed operations throw `WorldStorageError` with a `code` property: `QUOTA_EXCEEDED` | `LIMIT_EXCEEDED` | `ENTRY_TOO_LARGE` | `TYPE_MISMATCH` | `INVALID_KEY` | `NOT_IN_WORLD` | `RATE_LIMITED` | `UNAUTHORIZED` | `UNKNOWN`

```typescript
import { useWorldStorage, WorldStorageError } from '@xrift/world-components'

const storage = useWorldStorage()

// Shared KV (one shared value per world)
await storage.shared.set('event_phase', 'chapter-2')
const phase = await storage.shared.get('event_phase')
const visits = await storage.shared.increment('total_visits', 1)

// Per-player KV (writes: own values only; reads: others allowed)
await storage.player.set('coins', 340)
const coins = await storage.player.get('coins')
const otherScore = await storage.player.get('score', { userId })

// Error handling
try {
  await storage.player.increment('coins', 10)
} catch (e) {
  if (e instanceof WorldStorageError && e.code === 'UNAUTHORIZED') {
    // Guests cannot write
  }
}
```

---

### useServerClock(options?)

Hook providing a clock (server time) that agrees across every device in the instance (±40ms measured). Device `Date.now()` values differ by 0.1 to several seconds, so use this for anything requiring "the same moment": countdowns, simultaneous effects, video playback alignment, periodic animations everyone sees in phase.

**Arguments**: `options?.require: 'media' | 'motion'` — accuracy requirement, used to compute `trustworthy` (`media` = ±300ms for video/music alignment, `motion` = ±100ms for animations/effects)

**Returns**:

| Property | Type | Description |
|----------|------|-------------|
| `now` | `() => number` | Estimated server time (ms). **A function** — call from `useFrame` without re-rendering. Falls back to `Date.now()` before first sync |
| `uncertainty` | `number` | Error upper bound (ms), including aging. `Infinity` before first sync |
| `synced` | `boolean` | Whether sync is established. `false` while disconnected, but `now()` keeps returning the last estimate |
| `trustworthy` | `boolean` | Whether the `require` accuracy is met (same as `synced` if omitted) |
| `timeJumpCount` | `number` | Timeline jump count. Re-baseline delta-accumulating animations when it changes |
| `lastTimeJumpMs` | `number` | Most recent jump (ms); negative = jumped backward |

**Principles**:
- Prefer **stateless** rendering: write positions as `f(now())` every frame. Zero networking, late joiners match instantly, and time jumps self-heal with no handling needed
- Video sync: **never correct when the correction costs more than the error.** Dead band < 0.3s (and reset `playbackRate` to 1 inside it), absorb 0.3–5s via `playbackRate` ±5%, seek only when the target is already buffered, and give up (keep playing) when `trustworthy` is false
- **Not for fairness-critical judgments** (buzzers, finish lines): asymmetric-path error is undetectable client-side and constant within a session, so the same person wins every time. Adjudicate on the server
- In dev the default implementation is unsynced (`trustworthy` always false). Inject a fake via `<XRiftProvider serverClockImplementation={{ now: () => Date.now(), uncertainty: 10, synced: true, timeJumpCount: 0, lastTimeJumpMs: 0 }}>`
- Requires `@xrift/world-components` >= 0.47.0

```typescript
import { useServerClock } from '@xrift/world-components'
import { useFrame } from '@react-three/fiber'

// Everyone sees the same phase (stateless — no sync logic at all)
const { now } = useServerClock()
useFrame(() => {
  mesh.current.position.y = 1 + Math.sin(now() / 1000) * 0.5
})

// Gate on accuracy for media alignment
const clock = useServerClock({ require: 'media' })
if (clock.trustworthy) {
  const target = ((clock.now() - epoch) / 1000) % duration
}
```

---

## Components

### Interactable

A clickable interactive object. Automatically sets `LAYERS.INTERACTABLE` on child objects.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `onInteract` | `(id: string) => void` | Yes | Callback when clicked (receives the object ID) |
| `interactionText` | `string` | No | Text displayed on hover |
| `enabled` | `boolean` | No | Enable/disable interaction |
| `type` | `'button'` | No | Object type |

```typescript
import { Interactable } from '@xrift/world-components'

<Interactable
  id="my-button"
  onInteract={() => console.log('clicked')}
  interactionText="Click me"
>
  <mesh>
    <boxGeometry args={[1, 1, 0.2]} />
    <meshStandardMaterial color="blue" />
  </mesh>
</Interactable>
```

### Grabbable

Declares an object as grabbable. Players can grab it, float it in front of their view, and place
it somewhere else. Child meshes are put on `LAYERS.GRABBABLE` (layer 14); the grabbing foundation
(raycasting, carrying, committing) is provided by the platform, and `DevEnvironment` includes a
system so you can test it while developing.

`transform` and `onMove` use the **local space of the parent** you place the `Grabbable` in (the
same space as a normal `position` prop). Nesting it under a transformed parent group is safe — the
conversion to and from world space happens internally, so the release position never drifts.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `transform` | `GrabbableTransform` | Yes | Current pose, in the parent's local space. Applied to the root group, so write children around the origin |
| `onMove` | `(transform: GrabResultTransform) => void` | Yes | Called when the player releases it. Returns the same local space as `transform`, so store it in state and feed it back |
| `renderGhost` | `() => ReactNode` | No | Ghost shown while carrying (semi-transparent, no physics). Defaults to reusing `children` |
| `enabled` | `boolean` | No | Whether it can be grabbed (default: true) |

```typescript
import { useState } from 'react'
import { Grabbable, type GrabbableTransform } from '@xrift/world-components'

function GrabbableBall() {
  const [transform, setTransform] = useState<GrabbableTransform>({
    position: { x: 2, y: 0.5, z: -2 },
    rotation: { x: 0, y: 0, z: 0 },
  })

  return (
    <Grabbable
      id="ball"
      transform={transform}
      onMove={(next) => setTransform((prev) => ({ ...prev, ...next }))}
    >
      {/* children are written around the origin */}
      <mesh>
        <sphereGeometry args={[0.3]} />
        <meshStandardMaterial color="gold" />
      </mesh>
    </Grabbable>
  )
}
```

> **If children contain physics** (`RigidBody` etc.), you must pass a physics-free `renderGhost`.
> Otherwise `children` is reused as the ghost and the colliders overlap while carrying.

> Grabbing assumes desktop (pointer lock + center crosshair). In `DevEnvironment`: **G** to grab
> and place, mouse wheel to adjust distance, click to confirm, **Esc** to cancel.

### Seat

Declares an object as sittable. When a player sits, their view, pose, and body orientation follow
the seat; they stand up with **Space** (the A button in VR).

Place `Seat` as a group: **its origin is the seating surface (where the hips go) and its forward
direction is -Z**. Write children in coordinates relative to that surface.

The surface transform is derived from the **world matrix of wherever you placed the `Seat`, every
frame**. Nesting it under a moving vehicle, a turntable, or a tilted group just works — the player
follows along. You never pass the transform yourself.

**Props** (group properties such as `position` and `rotation` can be passed directly; `scale` cannot):
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `exitOffset` | `SeatExitOffset` | No | Where the player is placed on standing up (default: `{ forward: 0.6, right: 0, up: 0 }`) |
| `interactionText` | `string` | No | Text shown when aiming at the seat (default: `'座る'`) |
| `enabled` | `boolean` | No | Whether it can be used (default: true). Disabled automatically while someone else is seated |
| `onEnter` | `(occupant: SeatOccupant) => void` | No | Called when someone sits down (yourself or anyone else) |
| `onLeave` | `(occupant: SeatOccupant) => void` | No | Called when someone stands up (yourself or anyone else) |
| `driver` | `boolean` | No | Makes this the **driver seat** of the surrounding `Vehicle` (>= 0.52.0) |
| `onControlInput` | `(input: SeatControlInput, delta: number) => void` | No | Steering input, for seats that are **not** vehicles (turrets, swivel chairs) |

```typescript
import { Seat } from '@xrift/world-components'

const height = 0.45

// Place the Seat at the height of the seating surface
<Seat id="stool-1" position={[2, height, -3]} rotation={[0, Math.PI / 2, 0]}>
  {/* children are relative to the surface, so drop the box by half its height */}
  <mesh position={[0, -height / 2, 0]}>
    <boxGeometry args={[0.5, height, 0.5]} />
    <meshStandardMaterial color="saddlebrown" />
  </mesh>
</Seat>
```

> **Put the `Seat` at the surface height, not on the floor.** Aligning its origin with the floor
> makes players sit sunk into the ground.

> `scale` is not accepted. The surface is defined by position and orientation alone, and seated hip
> and eye heights come from the player's own avatar. Scale the `children` instead. If an ancestor
> group is scaled, the surface is still correct but `exitOffset` distances stay in world meters.

> `DevEnvironment` (0.53.0+) bundles a single-player seat system, so you can sit and try it
> during development: aim at a seat and click to sit, **Space** to stand up. Sitting in a `Vehicle`
> driver's seat lets you drive it with WASD.

> Requires `@xrift/world-components` >= 0.50.0

**To make a vehicle, use `Vehicle` (below) and mark its driver seat with `driver`.** `onControlInput`
is for seats that are not vehicles — a turret, a swivel chair, a crane — where you want the steering
input but own no vehicle transform.

```typescript
<Seat
  id="turret-1"
  position={[0, 0.5, 0]}
  onControlInput={(input, delta) => {
    // only fires while the local player is sitting here
    barrelYaw.current -= input.right * TURN_RATE * delta
  }}
/>
```

`SeatControlInput` gives you **which way the player wants to move**, not how far. Whether `right` means steering or strafing is for you to decide.

> **Prefer this over raw `keydown` listeners.** It arrives the same way in VR and on mobile (thumbsticks, virtual joystick), and it is limited to the person seated — with raw keys, someone who is not aboard can press W and move things on their own screen, drifting out of sync with everyone else.

### Vehicle

A rideable vehicle. Put `Seat`s inside it and mark one with `driver` to make it drivable.

**`Vehicle` owns the transform.** You write only *how you want it to move*, in `onDrive`; the
syncing is handled for you.

- On the driver's client: it moves by what `onDrive` writes, and that pose is what gets synced
- On everyone else's client: `onDrive` is not called; the arriving pose is applied instead

The whole body moves, so **empty passenger seats end up in the right place too**, and exactly one
pose is synced per vehicle.

**Props** (group properties such as `position` and `rotation` can be passed directly):
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier for the vehicle |
| `onDrive` | `(input: SeatControlInput, delta: number, vehicle: THREE.Group) => void` | No | Moves the vehicle. Called every frame **only while the local player is in the driver seat** |

`vehicle` is the three.js `Group` itself. Writing to it moves the vehicle:

- `vehicle.rotateY(rad)` — turn
- `vehicle.translateZ(-distance)` — drive toward **the body's own forward**, so slopes work for free
- `vehicle.quaternion` — set it directly for slopes, banking, even loops

```typescript
import { Seat, Vehicle } from '@xrift/world-components'

const SPEED = 3       // m/s
const TURN_RATE = 1.8 // rad/s

<Vehicle
  id="cart-1"
  position={[0, 0, -5]}
  onDrive={(input, delta, vehicle) => {
    // drive along the body's forward axis, so a slope is followed automatically
    vehicle.translateZ(-input.forward * SPEED * delta)
    // turn relative to where it is already facing
    vehicle.rotateY(-input.right * TURN_RATE * delta)
  }}
>
  {/* the body, in the vehicle's local coordinates */}
  <mesh position={[0, 0.25, 0]}>
    <boxGeometry args={[1.2, 0.3, 2]} />
    <meshStandardMaterial color="tomato" />
  </mesh>

  {/* the driver seat */}
  <Seat id="cart-1-driver" driver position={[0, 0.45, -0.35]} exitOffset={{ forward: 0, right: -1.2 }}>
    <mesh><boxGeometry args={[0.5, 0.1, 0.5]} /><meshStandardMaterial color="steelblue" /></mesh>
  </Seat>

  {/* a passenger seat - no driver, moves with the body anyway */}
  <Seat id="cart-1-back" position={[0, 0.45, 0.55]} exitOffset={{ forward: 0, right: 1.2 }}>
    <mesh><boxGeometry args={[0.5, 0.1, 0.5]} /><meshStandardMaterial color="seagreen" /></mesh>
  </Seat>
</Vehicle>
```

> **Use `translateZ` / `rotateY`, not world-axis arithmetic.** Adding to `position.x` / `position.z`
> by a heading angle throws the tilt away, so the vehicle floats above or sinks into a slope.

> **Do not drive the vehicle from props.** `position` and `rotation` set where it starts; after that
> the transform belongs to `Vehicle` (`onDrive` on the driver's client, the arriving pose on
> everyone else's). Re-rendering with a *changed* `position` fights whichever one is in charge.

> **Hold vehicle state as local state on the driver's client.** Do not sync it with
> `useInstanceState` — the pose is already synced for you, and doing both means managing it twice.

> **`driver` only means something inside a `Vehicle`.** On a `Seat` outside one it is ignored (with a
> console warning) and the seat behaves as an ordinary chair.

> Where the vehicle was parked is remembered for the instance, so someone who joins later sees it
> where it was left, not back at its starting `position`.

> Requires `@xrift/world-components` >= 0.52.0

### SpawnPoint

Player spawn location.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `position` | `[number, number, number]` | No | Spawn position |
| `yaw` | `number` | No | Facing direction (0-360 degrees) |

```typescript
import { SpawnPoint } from '@xrift/world-components'

<SpawnPoint position={[0, 0, 0]} yaw={180} />
```

### Mirror

Reflective surface component.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `size` | `[number, number]` | No | Size |
| `color` | `number` | No | Reflection color (default: 0xcccccc) |
| `textureResolution` | `number` | No | Texture resolution |
| `lodDistance` | `number` | No | Distance in meters to switch to envMap-based pseudo-mirror (default: 10) |
| `reflectionInterval` | `number` | No | Reflection texture update interval (once every N frames, default: 2) |

```typescript
import { Mirror } from '@xrift/world-components'

<Mirror position={[0, 1.5, -3]} size={[2, 3]} />
```

### VideoPlayer

Video player component with UI controls (play/pause, progress bar, volume, URL input). VR-compatible.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `url` | `string` | No | Video URL |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `width` | `number` | No | Width |
| `playing` | `boolean` | No | Playing state |
| `volume` | `number` | No | Volume |
| `sync` | `'global' \| 'local'` | No | Sync mode (default: 'global') |

```typescript
import { VideoPlayer } from '@xrift/world-components'

<VideoPlayer
  id="main-video"
  url="https://example.com/video.mp4"
  position={[0, 2, -5]}
  width={4}
/>
```

### Portal

A portal component for navigating to another instance. Displays the target instance's world thumbnail, name, and a vortex shader effect. When a player steps onto the pedestal, a confirmation modal is shown before navigating.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `instanceId` | `string` | Yes | Target instance ID |
| `position` | `[number, number, number]` | No | Position (default: `[0, 0, 0]`) |
| `rotation` | `[number, number, number]` | No | Rotation (default: `[0, 0, 0]`) |
| `disabled` | `boolean` | No | Disable portal navigation (default: false) |

```typescript
import { Portal } from '@xrift/world-components'

<Portal
  instanceId="target-instance-id"
  position={[5, 0, 0]}
/>
```

### BillboardY

Y-axis billboard component. Wraps children in a group that rotates on the Y-axis only to face the camera. Unlike drei's `<Billboard>` which rotates on all axes, this keeps the "up" direction intact — ideal for flames, particles, name plates, and signage.

Works correctly with Mirror (Reflector) — objects display with the correct orientation in reflections.

**Props**: Same as `<group>` (position, rotation, scale, etc.)

```typescript
import { BillboardY } from '@xrift/world-components'

<BillboardY position={[0, 2, 0]}>
  <mesh>
    <planeGeometry args={[1, 1.5]} />
    <meshBasicMaterial map={fireTexture} />
  </mesh>
</BillboardY>
```

### ScreenShareDisplay

Screen share display component.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `width` | `number` | No | Width |
| `targetFps` | `number` | No | Texture update FPS limit for low-spec devices (default: unlimited) |
| `placeholderImageUrl` | `string` | No | Image URL shown while no screen is being shared (contain-fit; falls back to background color and guide text on load failure). The image is loaded as a WebGL texture, so the URL must allow CORS |

```typescript
import { ScreenShareDisplay } from '@xrift/world-components'

<ScreenShareDisplay id="screen-share" position={[0, 2, -3]} width={4} placeholderImageUrl="https://example.com/placeholder.png" />
```

### Skybox

Gradient sky background.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `topColor` | `number` | No | Top color (default: 0x87ceeb) |
| `bottomColor` | `number` | No | Bottom color (default: 0xffffff) |
| `offset` | `number` | No | Gradient start position (default: 0) |
| `exponent` | `number` | No | Gradient range (default: 1) |

```typescript
import { Skybox } from '@xrift/world-components'

<Skybox topColor={0x87ceeb} bottomColor={0xffffff} />
```

### VideoScreen

Low-level video screen without UI controls. Use `VideoPlayer` for a full-featured player.

With `sync='global'` (the default), playback is kept in sync across the instance using an
**anchor**: the synced state carries `currentTime` together with `serverTime`, meaning "at server
time `serverTime`, the playback position was `currentTime`". Each client computes its own target
position from that, so **someone who joins later catches up with no extra communication**. Small
drift is corrected by nudging playback speed rather than seeking, so viewers rarely see a jump.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique screen ID |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `scale` | `[number, number]` | No | Size [width, height] |
| `url` | `string` | No | Video URL |
| `playing` | `boolean` | No | Playing state (default: true) |
| `currentTime` | `number` | No | Playback position in seconds |
| `sync` | `'global' \| 'local'` | No | Sync mode (default: 'global') |
| `muted` | `boolean` | No | Muted state (default: false) |
| `volume` | `number` | No | Volume 0-1 (default: 1) |

```typescript
import { VideoScreen } from '@xrift/world-components'

<VideoScreen id="bg-video" url="https://example.com/video.mp4" scale={[4, 2.25]} />
```

> If you write the synced state yourself instead of letting `VideoScreen` manage it, stamp
> `serverTime` with the shared clock (`useServerClock`), never `Date.now()`. Device clocks differ
> from each other by 0.1 to several seconds (a Quest was measured 0.66s off), and the anchor would
> place everyone at a different position.

> Anchor-based catch-up requires `@xrift/world-components` >= 0.48.0

### LiveVideoPlayer

HLS live stream video player with controls. Height is auto-calculated at 16:9 ratio.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique identifier |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `width` | `number` | No | Screen width (default: 4) |
| `url` | `string` | No | HLS stream URL (.m3u8) |
| `playing` | `boolean` | No | Initial playing state (default: false) |
| `volume` | `number` | No | Initial volume 0-1 (default: 1) |
| `sync` | `'global' \| 'local'` | No | Sync mode (default: 'global') |

```typescript
import { LiveVideoPlayer } from '@xrift/world-components'

<LiveVideoPlayer id="live-stream" url="https://example.com/live/stream.m3u8" position={[0, 2, -5]} width={6} />
```

### TextInput

3D text input component. Wraps child objects to make them trigger a text input prompt.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `id` | `string` | Yes | Unique input ID |
| `children` | `ReactNode` | Yes | 3D objects (trigger area) |
| `placeholder` | `string` | No | Placeholder text |
| `maxLength` | `number` | No | Maximum character count |
| `value` | `string` | No | Controlled value |
| `onSubmit` | `(value: string) => void` | No | Submit callback |
| `interactionText` | `string` | No | Text displayed on hover |
| `disabled` | `boolean` | No | Disable input |

```typescript
import { TextInput } from '@xrift/world-components'

<TextInput id="name-input" placeholder="Enter your name" onSubmit={setName}>
  <mesh><planeGeometry args={[2, 0.5]} /><meshStandardMaterial color="white" /></mesh>
</TextInput>
```

### TagBoard

Tag selection board where users can select tags displayed above their avatar.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `instanceStateKey` | `string` | Yes | Instance state key for multi-board identification |
| `tags` | `Tag[]` | No | Tag definitions (uses defaults if omitted) |
| `columns` | `number` | No | Display columns (default: 3) |
| `title` | `string` | No | Board title |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `scale` | `number` | No | Overall scale |

```typescript
import { TagBoard } from '@xrift/world-components'

<TagBoard instanceStateKey="role-tags" position={[0, 1.5, -3]} />
```

### Video180Sphere

180-degree VR video player rendered on a hemisphere.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `url` | `string` | Yes | 180-degree video URL |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `scale` | `number \| [number, number, number]` | No | Scale |
| `playing` | `boolean` | No | Playing state (default: true) |
| `muted` | `boolean` | No | Muted state |
| `volume` | `number` | No | Volume 0-1 (default: 1) |
| `radius` | `number` | No | Hemisphere radius |
| `segments` | `number` | No | Geometry resolution |
| `loop` | `boolean` | No | Loop playback |
| `placeholderColor` | `string` | No | Pre-load placeholder color |
| `onEnded` | `() => void` | No | Playback ended callback |
| `onLoadedMetadata` | `(event: { duration: number }) => void` | No | Metadata loaded callback |
| `onProgress` | `(event: { currentTime: number }) => void` | No | Progress update callback |

```typescript
import { Video180Sphere } from '@xrift/world-components'

<Video180Sphere url="https://example.com/vr-180.mp4" radius={5} loop />
```

### DevEnvironment

Development environment wrapper. Provides physics, camera, crosshair, first-person navigation, and a single-player seat system (sit on `Seat` targets, drive `Vehicle` from the driver's seat). Not needed in production.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `children` | `ReactNode` | Yes | World content |
| `camera` | `CameraConfig` | No | Camera settings |
| `moveSpeed` | `number` | No | Movement speed (default: 5.0) |
| `shadows` | `boolean` | No | Enable shadows (default: true) |
| `spawnPosition` | `[number, number, number]` | No | Spawn position (default: [0.11, 1.6, 7.59]) |
| `respawnThreshold` | `number` | No | Respawn height threshold (default: -10) |
| `physicsConfig` | `PhysicsConfig` | No | Physics settings |

```typescript
import { DevEnvironment } from '@xrift/world-components'

<DevEnvironment physicsConfig={{ gravity: 9.81, allowInfiniteJump: false }}>
  <World />
</DevEnvironment>
```

### EntryLogBoard

Displays a log of user join/leave events.

**Props**:
| Prop | Type | Required | Description |
|------|------|----------|-------------|
| `stateNamespace` | `string` | No | Instance state key for multi-board identification |
| `maxEntries` | `number` | No | Maximum display entries |
| `formatTimestamp` | `(timestampMs: number) => string` | No | Timestamp format function (receives epoch ms) |
| `displayNameFallback` | `string` | No | Fallback when display name unavailable |
| `labels` | `Partial<Labels>` | No | Customize join/leave labels |
| `colors` | `Partial<Colors>` | No | Customize colors |
| `position` | `[number, number, number]` | No | Position |
| `rotation` | `[number, number, number]` | No | Rotation |
| `scale` | `number` | No | Overall scale |
| `onJoin` | `(entry: LogEntry) => void` | No | Join event callback |
| `onLeave` | `(entry: LogEntry) => void` | No | Leave event callback |

```typescript
import { EntryLogBoard } from '@xrift/world-components'

<EntryLogBoard position={[3, 1.5, -2]} maxEntries={10} />
```

---

## Constants

### LAYERS (Three.js Layer Constants)

Three.js cameras and objects have 32 layers (0-31). A camera only renders objects belonging to its enabled layers.

| Constant | Value | Purpose |
|----------|-------|---------|
| `LAYERS.DEFAULT` | 0 | Default layer (all objects belong to this initially) |
| `LAYERS.FIRST_PERSON_ONLY` | 9 | First-person view only (for VRM headless copy) |
| `LAYERS.THIRD_PERSON_ONLY` | 10 | Third-person view only (for other players and mirrors) |
| `LAYERS.INTERACTABLE` | 11 | Interactable objects (for Raycast detection) |

```typescript
import { LAYERS } from '@xrift/world-components'
```

**How It Works**:
- The `Interactable` component automatically sets `LAYERS.INTERACTABLE` on child objects
- In production, the frontend performs Raycasts on the `LAYERS.INTERACTABLE` layer to detect interactions
- In the dev environment (`dev.tsx`), you need to set the Raycaster layer to `LAYERS.INTERACTABLE` to test interactions

```typescript
// Detect only interactable objects with Raycaster
const raycaster = new Raycaster()
raycaster.layers.set(LAYERS.INTERACTABLE) // Only hits objects on layer 11
```
