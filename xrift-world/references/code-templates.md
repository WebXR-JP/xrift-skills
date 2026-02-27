# コードテンプレート

XRift ワールド開発でよく使うパターンのテンプレート集。

## GLBモデル読み込み

```typescript
import { useXRift } from '@xrift/world-components'
import { useGLTF } from '@react-three/drei'
import { RigidBody } from '@react-three/rapier'

export const MyModel = () => {
  const { baseUrl } = useXRift()
  const { scene } = useGLTF(`${baseUrl}model.glb`)

  return (
    <RigidBody type="fixed">
      <primitive object={scene} castShadow receiveShadow />
    </RigidBody>
  )
}
```

## テクスチャ読み込み（単体）

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'

export const TexturedMesh = () => {
  const { baseUrl } = useXRift()
  const texture = useTexture(`${baseUrl}albedo.png`)

  return (
    <mesh>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial map={texture} />
    </mesh>
  )
}
```

## 複数テクスチャ（PBR）

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'

export const PBRMaterial = () => {
  const { baseUrl } = useXRift()
  const [albedo, normal, roughness] = useTexture([
    `${baseUrl}albedo.png`,
    `${baseUrl}normal.png`,
    `${baseUrl}roughness.png`,
  ])

  return (
    <meshStandardMaterial
      map={albedo}
      normalMap={normal}
      roughnessMap={roughness}
    />
  )
}
```

## Skybox（360度パノラマ背景）

```typescript
import { useXRift } from '@xrift/world-components'
import { useTexture } from '@react-three/drei'
import { BackSide } from 'three'

export const Skybox = ({ radius = 500 }) => {
  const { baseUrl } = useXRift()
  const texture = useTexture(`${baseUrl}skybox.jpg`)

  return (
    <mesh>
      <sphereGeometry args={[radius, 60, 40]} />
      <meshBasicMaterial map={texture} side={BackSide} />
    </mesh>
  )
}
```

## インタラクション + 状態同期

```typescript
import { Interactable, useInstanceState } from '@xrift/world-components'

export const InteractiveButton = ({ id }: { id: string }) => {
  // useInstanceState: 全ユーザー間で同期される状態
  const [clickCount, setClickCount] = useInstanceState(`${id}-count`, 0)

  return (
    <Interactable
      id={id}
      onInteract={() => setClickCount((prev) => prev + 1)}
      interactionText={`クリック回数: ${clickCount}`}
    >
      <mesh>
        <boxGeometry args={[1, 1, 0.2]} />
        <meshStandardMaterial color={clickCount > 0 ? 'green' : 'gray'} />
      </mesh>
    </Interactable>
  )
}
```

## アニメーション（useFrame）

```typescript
import { useRef } from 'react'
import { useFrame } from '@react-three/fiber'
import type { Mesh } from 'three'

export const RotatingCube = ({ speed = 1 }) => {
  const meshRef = useRef<Mesh>(null)

  useFrame((_, delta) => {
    if (meshRef.current) {
      meshRef.current.rotation.y += delta * speed
    }
  })

  return (
    <mesh ref={meshRef}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="hotpink" />
    </mesh>
  )
}
```

## テレポート（Interactable方式）

クリックでテレポートするパターン。

```typescript
import { useTeleport, Interactable } from '@xrift/world-components'

export const TeleportButton = () => {
  const { teleport } = useTeleport()
  return (
    <Interactable
      id="tp-button"
      onInteract={() => teleport({ position: [50, 0, 30], yaw: 180 })}
      interactionText="テレポート"
    >
      <mesh>
        <boxGeometry />
        <meshStandardMaterial color="purple" />
      </mesh>
    </Interactable>
  )
}
```

## テレポート（sensor方式）

触れたらテレポートするワープゾーンパターン。

```typescript
import { useCallback } from 'react'
import { useTeleport } from '@xrift/world-components'
import { RigidBody } from '@react-three/rapier'

export const TeleportZone = () => {
  const { teleport } = useTeleport()
  const handleEnter = useCallback(() => {
    teleport({ position: [0, 0.5, 50], yaw: 0 })
  }, [teleport])

  return (
    <RigidBody type="fixed" sensor onIntersectionEnter={handleEnter}>
      <mesh>
        <cylinderGeometry args={[1.2, 1.2, 1, 32]} />
        <meshBasicMaterial visible={false} />
      </mesh>
    </RigidBody>
  )
}
```

**注意**: sensor 方式でワープゾーンを作る場合、テレポート先が別のポータルと重ならないようにすること（着地即ワープのループになる）

## ユーザー位置追跡

```typescript
import { useFrame } from '@react-three/fiber'
import { useUsers } from '@xrift/world-components'

export const UserTracker = () => {
  const { remoteUsers, getMovement, getLocalMovement } = useUsers()

  useFrame(() => {
    // 自分の位置
    const myMovement = getLocalMovement()
    console.log('My position:', myMovement.position)

    // 他ユーザーの位置
    remoteUsers.forEach((user) => {
      const movement = getMovement(user.socketId)
      if (movement) {
        console.log(`${user.displayName}:`, movement.position)
      }
    })
  })

  return null
}
```
