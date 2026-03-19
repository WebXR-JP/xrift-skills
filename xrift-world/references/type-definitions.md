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

## VideoState

Video screen synchronized state (used internally by `VideoScreen`).

```typescript
interface VideoState {
  url: string
  isPlaying: boolean
  currentTime: number
  serverTime: number
}
```

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
