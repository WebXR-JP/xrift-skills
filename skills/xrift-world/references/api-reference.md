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

Hook for getting the unique ID of a placed item. Each placement gets a different ID, even for the same item type.

**Returns**: `{ id: string }`

| Property | Type | Description |
|----------|------|-------------|
| `id` | `string` | Unique placement ID (UUID) |

```typescript
import { useItem, useInstanceState } from '@xrift/world-components'

const { id } = useItem()
// Use id to scope per-placement state
const [votes, setVotes] = useInstanceState(`votes-${id}`, 0)
```

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

Development environment wrapper. Provides physics, camera, crosshair, and first-person navigation. Not needed in production.

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
| `formatTimestamp` | `(date: Date) => string` | No | Timestamp format function |
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
