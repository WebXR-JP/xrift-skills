# 型定義

XRift ワールドで使用される主要な型定義。

## User

ユーザー情報を表す型。`useUsers()` フックから取得される。

```typescript
interface User {
  id: string           // 認証ユーザーID
  socketId: string     // ソケット接続ID
  displayName: string  // 表示名
  avatarUrl: string | null
  isGuest: boolean
}
```

## PlayerMovement

プレイヤーの移動状態を表す型。`getMovement()` / `getLocalMovement()` から取得される。

```typescript
interface PlayerMovement {
  position: { x: number; y: number; z: number }
  direction: { x: number; z: number }  // 移動方向（正規化）
  horizontalSpeed: number              // XZ平面速度
  verticalSpeed: number                // Y軸速度
  rotation: { yaw: number; pitch: number }
  isGrounded: boolean
  isJumping: boolean
  isInVR?: boolean
  vrTracking?: VRTrackingData  // VRモード時のみ
}
```

## VRTrackingData

VRモード時のトラッキング情報。`PlayerMovement.vrTracking` から取得される。

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

## TeleportDestination

テレポート先を表す型。`useTeleport()` の `teleport()` に渡す。

```typescript
interface TeleportDestination {
  position: [number, number, number]
  yaw?: number  // 度数法（0-360）、省略時は現在の向きを維持
}
```
