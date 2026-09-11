# Type Definitions

Core type definitions used in XRift worlds.

## User

Represents user information. Retrieved from the `useUsers()` hook.

```typescript
interface User {
  id: string           // Authenticated user ID
  socketId: string     // Socket connection ID
  displayName: string  // Display name
  userIconUrl: string | null
  isGuest: boolean
}
```

## PlayerMovement

Represents a player's movement state. Retrieved from `getMovement()` / `getLocalMovement()`.

```typescript
interface PlayerMovement {
  position: { x: number; y: number; z: number }
  direction: { x: number; z: number }  // Movement direction (normalized)
  horizontalSpeed: number              // XZ-plane speed
  verticalSpeed: number                // Y-axis speed
  rotation: { yaw: number; pitch: number }
  isGrounded: boolean
  isJumping: boolean
  isInVR?: boolean
  vrTracking?: VRTrackingData  // Only present in VR mode
}
```

## AvatarHeight

Represents an avatar's height information. Retrieved from `getAvatarHeight()` / `getLocalAvatarHeight()` via the `useUsers()` hook.

```typescript
interface AvatarHeight {
  height: number    // Full height of the avatar (meters)
  eyeHeight: number // Height from ground to the avatar's eye position (meters)
}
```

## VRTrackingData

VR mode tracking information. Retrieved from `PlayerMovement.vrTracking`.

```typescript
interface VRTrackingData {
  head: { yaw: number; pitch: number }
  leftHand: { position: Position3D; rotation: Rotation3D }
  rightHand: { position: Position3D; rotation: Rotation3D }
  hipsPositionDelta: Position3D
  movementDirection: 'forward' | 'backward' | 'left' | 'right' | 'idle'
  isHandTracking?: boolean
}
```

## ConfirmOptions

Options passed to `requestConfirm()` from the `useConfirm()` hook.

```typescript
interface ConfirmOptions {
  title?: string    // Dialog title
  message: string   // Message displayed to the user
  confirmLabel?: string  // Label for the confirm button
  cancelLabel?: string   // Label for the cancel button
}
```

## ConfirmContextValue

Context value provided by `ConfirmContext`. Retrieved via the `useConfirm()` hook.

```typescript
interface ConfirmContextValue {
  requestConfirm: (options: ConfirmOptions) => Promise<boolean>
}
```

## FileInputRequest

Options passed to `requestFileInput()` from the `useFileInput()` hook.

```typescript
interface FileInputRequest {
  id: string                              // Unique identifier for the input
  accept?: string                         // Accepted file types (e.g. '.vrm', 'image/*')
  multiple?: boolean                      // Allow multiple file selection
  maxSize?: number                        // Maximum file size in bytes
  onSelect: (files: File[]) => void       // Callback when files are selected
  onCancel?: () => void                   // Callback when cancelled
  onError?: (error: FileInputError) => void // Callback on error
}
```

## FileInputError

Error information returned to `onError` callback.

```typescript
type FileInputErrorType = 'file_too_large' | 'invalid_type'

interface FileInputError {
  type: FileInputErrorType
  message: string
}
```

## FileInputContextValue

Context value provided by `FileInputContext`. Retrieved via the `useFileInput()` hook.

```typescript
interface FileInputContextValue {
  requestFileInput: (request: FileInputRequest) => void
  isActive: boolean
}
```

## SharedFileInfo

Information about a shared file uploaded to an instance.

```typescript
interface SharedFileInfo {
  id: string                                // Unique file ID
  fileName: string                          // Original file name
  contentType: string                       // MIME type (e.g. 'image/png')
  fileSize: number                          // File size in bytes
  publicUrl: string                         // Public URL for accessing the file
  locked: boolean                           // Whether the file is locked (deletion protection)
  description: string | null                // Description text (up to 500 characters)
  metadata: Record<string, string> | null   // Flat key-value metadata (up to 20 entries)
  createdAt: string                         // Creation date (ISO 8601)
}
```

## UploadSharedFileOptions

Optional information attached at upload time. Passed as the third argument of `uploadSharedFile`.

```typescript
interface UploadSharedFileOptions {
  description?: string               // Description text (up to 500 characters)
  metadata?: Record<string, string>  // Flat key-value metadata (up to 20 entries, keys 1-64 chars, values up to 500 chars)
}
```

## UpdateSharedFileParams

Update payload for `updateSharedFile`. Pass `null` to clear `description` / `metadata`.

```typescript
interface UpdateSharedFileParams {
  fileName?: string
  description?: string | null               // null clears the field
  metadata?: Record<string, string> | null  // null clears the field
}
```

## SharedFileContextValue

Context value provided by `SharedFileContext`. Retrieved via the `useSharedFile()` hook.

```typescript
interface SharedFileContextValue {
  uploadSharedFile: (
    file: File,
    onProgress?: (progress: number) => void,
    options?: UploadSharedFileOptions,
  ) => Promise<SharedFileInfo>
  getSharedFiles: () => Promise<SharedFileInfo[]>
  // Set the lock state (deletion protection). Locked files cannot be deleted or updated
  setSharedFileLock: (fileId: string, locked: boolean) => Promise<SharedFileInfo>
  // Update fileName / description / metadata. Rejected while locked
  updateSharedFile: (fileId: string, updates: UpdateSharedFileParams) => Promise<SharedFileInfo>
  // Delete a file. Rejected while locked
  deleteSharedFile: (fileId: string) => Promise<void>
}
```

## WorldStorageContextValue / SharedWorldStorage / PlayerWorldStorage

World-scoped persistent KV storage. Retrieved via the `useWorldStorage()` hook.

```typescript
interface WorldStorageContextValue {
  shared: SharedWorldStorage  // Shared KV (one shared value per world)
  player: PlayerWorldStorage  // Per-player KV
}

interface SharedWorldStorage {
  get: (key: string) => Promise<unknown>                       // undefined if missing
  list: () => Promise<WorldStorageEntry[]>
  set: (key: string, value: unknown) => Promise<void>
  increment: (key: string, delta: number) => Promise<number>   // atomic add, returns new value
  delete: (key: string) => Promise<void>                       // idempotent
}

interface PlayerWorldStorage {
  get: (key: string, options?: { userId?: string }) => Promise<unknown>  // userId: read another user's value
  list: (options?: { userId?: string }) => Promise<WorldStorageEntry[]>
  set: (key: string, value: unknown) => Promise<void>                    // own value only
  increment: (key: string, delta: number) => Promise<number>             // own value only
  delete: (key: string) => Promise<void>                                 // own value only
}

interface WorldStorageEntry {
  key: string
  value: unknown
}
```

## WorldStorageError

Error thrown when a World Storage operation fails.

```typescript
type WorldStorageErrorCode =
  | 'QUOTA_EXCEEDED'   // Total world capacity (10MB) exceeded
  | 'LIMIT_EXCEEDED'   // Key count limit exceeded (shared: 256 / per user: 64)
  | 'ENTRY_TOO_LARGE'  // Entry size limit (100KB) exceeded
  | 'TYPE_MISMATCH'    // increment target is not a number
  | 'INVALID_KEY'      // Key does not match /^[A-Za-z0-9_.\-:]{1,128}$/
  | 'NOT_IN_WORLD'     // Writing while not in a world instance
  | 'RATE_LIMITED'     // Write rate limit exceeded (30/min per user)
  | 'UNAUTHORIZED'     // Writing as an unauthenticated guest
  | 'UNKNOWN'

class WorldStorageError extends Error {
  readonly code: WorldStorageErrorCode
}
```

## PortalProps

Props for the `Portal` component.

```typescript
interface PortalProps {
  instanceId: string
  position?: [number, number, number]  // Default: [0, 0, 0]
  rotation?: [number, number, number]  // Default: [0, 0, 0]
  disabled?: boolean                   // Default: false
}
```

## InstanceInfo

Represents instance information. Retrieved from `useInstance()`.

```typescript
interface InstanceInfo {
  id: string
  name: string
  description: string | null
  currentUsers: number
  maxCapacity: number
  isPublic: boolean
  allowGuests: boolean
  owner?: {
    id: string
    displayName: string
    userIconUrl?: string | null
  }
  world: WorldInfo
}
```

## WorldInfo

Represents world information. Retrieved from `useWorld()` or `InstanceInfo.world`.

```typescript
interface WorldInfo {
  id: string
  name: string
  description: string | null
  thumbnailUrl: string | null
  isPublic: boolean
  instanceCount: number
  totalVisitCount: number
  uniqueVisitorCount: number
  favoriteCount: number
  owner?: {
    id: string
    displayName: string
    userIconUrl?: string | null
  }
  permissions?: {
    allowedDomains: string[]   // Allowed external domains for network access
    allowedCodeRules: string[] // Relaxed code security rules (see xrift.json permissions in SKILL.md)
  }
}
```

## InstanceContextValue

Context value provided by `InstanceContext`. Injected by the platform to provide instance data fetching and navigation.

```typescript
interface InstanceContextValue {
  getInstanceInfo: (instanceId: string) => Promise<InstanceInfo>
  navigateToInstance: (instanceId: string) => void
}
```

## WorldContextValue

Context value provided by `WorldContext`. Injected by the platform to provide world data fetching.

```typescript
interface WorldContextValue {
  getWorldInfo: (worldId: string) => Promise<WorldInfo>
}
```

## TeleportDestination

Represents a teleport destination. Passed to `teleport()` from `useTeleport()`.

```typescript
interface TeleportDestination {
  position: [number, number, number]
  yaw?: number  // Degrees (0-360). If omitted, the player's current facing direction is preserved.
}
```

## Tag

Tag definition for `TagBoard`.

```typescript
interface Tag {
  id: string
  label: string
  color: string
}
```

## GrabbableTransform / GrabResultTransform

Pose of a `Grabbable` object. Both are in the **local space of the parent** you placed the
`Grabbable` in (the same space as a normal `position` prop).

```typescript
interface GrabbableTransform {
  position: { x: number; y: number; z: number }
  rotation: { x: number; y: number; z: number }  // Euler angles in radians
  scale?: number                                  // uniform scale (default: 1)
}

interface GrabResultTransform {
  position: { x: number; y: number; z: number }
  rotation: { x: number; y: number; z: number }
}
```

## SeatExitOffset

Where a player is placed when they stand up from a `Seat`. Distances are in **world meters**, so
they are not affected by a scaled ancestor group.

```typescript
interface SeatExitOffset {
  forward?: number  // toward the seat's front (-Z). Default 0.6
  right?: number    // to the seat's right. Default 0
  up?: number       // along world up. Default 0
}
```

`forward` and `right` are rotated by the seat's **horizontal facing (yaw) only**. `up` is world up:
using the seat's own up axis would bury the player in the ground when they leave a vehicle that is
upside down mid-loop.

## ItemPlacer

Who placed an item (`useItem().placedBy`).

```typescript
interface ItemPlacer {
  id: string                  // the placer's userId (always present)
  displayName: string | null  // null if the profile cannot be resolved
  avatarUrl: string | null    // null if it cannot be resolved
  isLocalUser: boolean
}
```

`displayName` and `avatarUrl` are resolved on demand, so they become `null` once the placer leaves
the instance. Use `id` when you need a stable key.

## VideoState

Video screen synchronized state (used internally by `VideoScreen`).

`currentTime` and `serverTime` form an **anchor**: "at server time `serverTime`, the playback
position was `currentTime`". Each client computes its own target position from that, so someone who
joins later catches up with no extra communication.

```typescript
interface VideoState {
  url: string
  isPlaying: boolean
  currentTime: number  // playback position at the anchor (seconds)
  serverTime: number   // the anchor's server time (ms), from useServerClock
}
```

> Stamp `serverTime` with the shared clock (`useServerClock`), never `Date.now()`. Device clocks
> differ from each other by 0.1 to several seconds.

## LogEntry

Entry log record for `EntryLogBoard`.

```typescript
type LogType = 'join' | 'leave'

interface LogEntry {
  id: string
  type: LogType
  userId: string
  displayName: string
  avatarUrl: string | null
  timestamp: string  // Formatted timestamp
}
```

## Labels / Colors (EntryLogBoard)

```typescript
interface Labels {
  join: string
  leave: string
}

interface Colors {
  join: string
  leave: string
  background: string
  text: string
}
```

## PhysicsConfig

Physics settings for `DevEnvironment`.

```typescript
interface PhysicsConfig {
  gravity?: number             // Default: 9.81
  allowInfiniteJump?: boolean  // Default: true
}
```

## CameraConfig

Camera settings for `DevEnvironment`.

```typescript
interface CameraConfig {
  near?: number  // Default: 0.01
  far?: number   // Default: 1000
}
```

## VoiceVolumeOverrideContextValue

Context value for voice volume override functionality.

> Renamed from `AudioVolumeContextValue` in v0.34.0. Old name still works but is deprecated.

```typescript
interface VoiceVolumeOverrideContextValue {
  setOverride: (userId: string, volume: number) => void
  clearOverride: (userId: string) => void
  clearAll: () => void
  getOverrides: () => ReadonlyMap<string, number>
}
```

## ItemContextValue

Value returned by `useItem()`. Contains the unique placement ID for the item.

```typescript
interface ItemContextValue {
  id: string  // Unique placement ID (UUID)
}
```

## Position3D / Rotation3D

Basic 3D coordinate types.

```typescript
interface Position3D {
  x: number
  y: number
  z: number
}

interface Rotation3D {
  x: number
  y: number
  z: number
}
```


## ServerClockAccuracy / UseServerClockResult

Instance-wide shared clock (server time). Retrieved via the `useServerClock()` hook.

```typescript
type ServerClockAccuracy = 'media' | 'motion'
// media  = ±300ms (video / music playback alignment)
// motion = ±100ms (periodic animations, simultaneous effects)

interface UseServerClockResult {
  now: () => number         // Estimated server time (ms). A function — call from useFrame without re-render. Falls back to Date.now() before first sync
  uncertainty: number       // Error upper bound (ms), including aging over time. Infinity before first sync
  synced: boolean           // Whether sync is established. false while disconnected (now() keeps returning the last estimate)
  trustworthy: boolean      // Whether the accuracy required via options.require is met (same as synced if omitted)
  timeJumpCount: number     // Timeline jump count. Re-baseline delta-accumulating animations when this changes
  lastTimeJumpMs: number    // Most recent jump (ms); negative = jumped backward
}
```
