# @xrift/world-components API リファレンス

## フック

### useXRift()

アセットURL取得用フック。ワールド内のアセット読み込みに必須。

**戻り値**: `{ baseUrl: string }`

```typescript
import { useXRift } from '@xrift/world-components'

const { baseUrl } = useXRift()
// baseUrl は末尾に / を含む
// 正しい: `${baseUrl}model.glb`
// 間違い: `${baseUrl}/model.glb`
```

### useInstanceState(key, initialValue)

全ユーザー間で状態を同期するフック。ワールドインスタンス内の全プレイヤーで共有される。

**引数**:
| 引数 | 型 | 説明 |
|------|-----|------|
| `key` | `string` | 状態の一意キー |
| `initialValue` | `T` | 初期値 |

**戻り値**: `[T, (value: T | ((prev: T) => T)) => void]`

```typescript
import { useInstanceState } from '@xrift/world-components'

const [count, setCount] = useInstanceState('click-count', 0)
setCount(prev => prev + 1)
```

### useUsers()

ユーザー情報・位置取得フック。

**戻り値**:
| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `localUser` | `User` | 自分のユーザー情報 |
| `remoteUsers` | `User[]` | 他のユーザー一覧 |
| `getMovement(socketId)` | `(id: string) => PlayerMovement \| null` | 指定ユーザーの移動情報取得 |
| `getLocalMovement()` | `() => PlayerMovement` | 自分の移動情報取得 |

```typescript
import { useUsers } from '@xrift/world-components'

const { localUser, remoteUsers, getMovement, getLocalMovement } = useUsers()
```

### useSpawnPoint()

スポーン地点取得フック。

**戻り値**: `{ position: [number, number, number], yaw: number }`

```typescript
import { useSpawnPoint } from '@xrift/world-components'

const { position, yaw } = useSpawnPoint()
```

### useScreenShareContext()

画面共有状態フック。

**戻り値**:
| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `videoElement` | `HTMLVideoElement \| null` | 画面共有の映像要素 |
| `isSharing` | `boolean` | 共有中かどうか |
| `startScreenShare` | `() => void` | 画面共有開始 |
| `stopScreenShare` | `() => void` | 画面共有停止 |

```typescript
import { useScreenShareContext } from '@xrift/world-components'

const { videoElement, isSharing, startScreenShare, stopScreenShare } = useScreenShareContext()
```

### useTeleport()

プレイヤー瞬間移動フック。

**戻り値**: `{ teleport: (dest: TeleportDestination) => void }`

`TeleportDestination`: `{ position: [number, number, number], yaw?: number }`
- `yaw` は度数法（0-360）、省略時は現在の向きを維持

```typescript
import { useTeleport } from '@xrift/world-components'

const { teleport } = useTeleport()
teleport({ position: [50, 0, 30], yaw: 180 })
```

[useTeleport ドキュメント](https://docs.xrift.net/world-components/components/#useteleport)

---

## コンポーネント

### Interactable

クリック可能なインタラクティブオブジェクト。子オブジェクトに自動で `LAYERS.INTERACTABLE` を設定する。

**Props**:
| Prop | 型 | 必須 | 説明 |
|------|-----|------|------|
| `id` | `string` | 必須 | 一意の識別子 |
| `onInteract` | `() => void` | 必須 | クリック時のコールバック |
| `interactionText` | `string` | 任意 | ホバー時に表示するテキスト |
| `enabled` | `boolean` | 任意 | インタラクション有効/無効 |

```typescript
import { Interactable } from '@xrift/world-components'

<Interactable
  id="my-button"
  onInteract={() => console.log('clicked')}
  interactionText="クリック"
>
  <mesh>
    <boxGeometry args={[1, 1, 0.2]} />
    <meshStandardMaterial color="blue" />
  </mesh>
</Interactable>
```

### SpawnPoint

プレイヤー出現地点。

**Props**:
| Prop | 型 | 必須 | 説明 |
|------|-----|------|------|
| `position` | `[number, number, number]` | 任意 | 出現位置 |
| `yaw` | `number` | 任意 | 向き（0-360度） |

```typescript
import { SpawnPoint } from '@xrift/world-components'

<SpawnPoint position={[0, 0, 0]} yaw={180} />
```

### Mirror

反射面コンポーネント。

**Props**:
| Prop | 型 | 必須 | 説明 |
|------|-----|------|------|
| `position` | `[number, number, number]` | 任意 | 位置 |
| `rotation` | `[number, number, number]` | 任意 | 回転 |
| `size` | `[number, number]` | 任意 | サイズ |
| `color` | `string` | 任意 | 色 |
| `textureResolution` | `number` | 任意 | テクスチャ解像度 |

```typescript
import { Mirror } from '@xrift/world-components'

<Mirror position={[0, 1.5, -3]} size={[2, 3]} />
```

### VideoPlayer

UI付き動画再生コンポーネント。UIコントロール付き（再生/一時停止、進捗バー、音量調整、URL入力）、VR対応。

**Props**:
| Prop | 型 | 必須 | 説明 |
|------|-----|------|------|
| `id` | `string` | 必須 | 一意の識別子 |
| `url` | `string` | 必須 | 動画URL |
| `position` | `[number, number, number]` | 任意 | 位置 |
| `rotation` | `[number, number, number]` | 任意 | 回転 |
| `width` | `number` | 任意 | 横幅 |
| `playing` | `boolean` | 任意 | 再生状態 |
| `volume` | `number` | 任意 | 音量 |
| `sync` | `boolean` | 任意 | ユーザー間同期 |

```typescript
import { VideoPlayer } from '@xrift/world-components'

<VideoPlayer
  id="main-video"
  url="https://example.com/video.mp4"
  position={[0, 2, -5]}
  width={4}
/>
```

### ScreenShareDisplay

画面共有表示コンポーネント。

**Props**:
| Prop | 型 | 必須 | 説明 |
|------|-----|------|------|
| `id` | `string` | 必須 | 一意の識別子 |
| `position` | `[number, number, number]` | 任意 | 位置 |
| `rotation` | `[number, number, number]` | 任意 | 回転 |
| `width` | `number` | 任意 | 横幅 |

```typescript
import { ScreenShareDisplay } from '@xrift/world-components'

<ScreenShareDisplay id="screen-share" position={[0, 2, -3]} width={4} />
```

---

## 定数

### LAYERS（Three.js レイヤー定数）

Three.js のカメラとオブジェクトは 32 のレイヤー（0-31）を持ち、カメラは有効化されたレイヤーに属するオブジェクトのみをレンダリングする。

| 定数名 | 値 | 用途 |
|--------|-----|------|
| `LAYERS.DEFAULT` | 0 | デフォルトレイヤー（すべてのオブジェクトが初期状態で属する） |
| `LAYERS.FIRST_PERSON_ONLY` | 9 | 一人称視点のみ表示（VRM のヘッドレスコピー用） |
| `LAYERS.THIRD_PERSON_ONLY` | 10 | 三人称視点のみ表示（他プレイヤーやミラー用） |
| `LAYERS.INTERACTABLE` | 11 | インタラクト可能オブジェクト（Raycast 検出用） |

```typescript
import { LAYERS } from '@xrift/world-components'
```

**仕組み**:
- `Interactable` コンポーネントは子オブジェクトに自動で `LAYERS.INTERACTABLE` を設定する
- 本番環境ではフロントエンド側が `LAYERS.INTERACTABLE` レイヤーで Raycast を行いインタラクションを検出する
- 開発環境（`dev.tsx`）でインタラクションをテストする場合、Raycaster のレイヤーを `LAYERS.INTERACTABLE` に設定する必要がある

```typescript
// Raycaster でインタラクト可能オブジェクトのみを検出
const raycaster = new Raycaster()
raycaster.layers.set(LAYERS.INTERACTABLE) // レイヤー11のオブジェクトのみヒット
```
