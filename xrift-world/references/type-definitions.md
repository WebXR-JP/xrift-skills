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
    allowedCodeRules: string[] // Relaxed code security rules (see below)
  }
}
```

### allowedCodeRules Reference

Rules defined by `@xrift/code-security` analyzer. When a world requires relaxation of these rules, they appear in `permissions.allowedCodeRules`.

#### Dynamic Code Execution
| Rule | Description |
|------|-------------|
| `no-eval` | Allows `eval()` to execute strings as code |
| `no-new-function` | Allows `Function` constructor to dynamically generate code |
| `no-string-timeout` | Allows `setTimeout`/`setInterval` with string arguments |
| `no-javascript-blob` | Allows creating JavaScript Blobs for dynamic script execution |

#### Obfuscation
| Rule | Description |
|------|-------------|
| `no-obfuscation` | Allows obfuscated code patterns |

#### Network
| Rule | Description |
|------|-------------|
| `no-network-without-permission` | Allows network requests (fetch, WebSocket, etc.) |
| `no-unauthorized-domain` | Allows connections to domains not in `allowedDomains` |
| `no-rtc-connection` | Allows WebRTC peer connections |
| `no-external-import` | Allows importing JavaScript modules from external URLs |

#### Storage & Data
| Rule | Description |
|------|-------------|
| `no-storage-access` | Allows localStorage/sessionStorage access |
| `no-cookie-access` | Allows cookie read/write |
| `no-indexeddb-access` | Allows IndexedDB access |
| `no-storage-event` | Allows listening to storage events from other tabs |

#### DOM
| Rule | Description |
|------|-------------|
| `no-dangerous-dom` | Allows innerHTML and script element injection |

#### Browser APIs
| Rule | Description |
|------|-------------|
| `no-navigator-access` | Allows access to geolocation, camera, microphone, clipboard, etc. |

#### Global Pollution
| Rule | Description |
|------|-------------|
| `no-sensitive-api-override` | Allows overriding security-critical APIs (fetch, etc.) |
| `no-global-override` | Allows overriding global objects (window, document, etc.) |
| `no-prototype-pollution` | Allows modifying built-in object prototypes |

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
