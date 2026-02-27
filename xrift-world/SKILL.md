---
name: xrift-world
description: XRiftプラットフォーム用WebXRワールド制作ガイド。React Three Fiber + Rapier物理エンジン + @xrift/world-components APIのフック、コンポーネント、コードテンプレート、型定義。
---

# XRift World 制作ガイド

XRiftプラットフォーム用WebXRワールドを作成・修正する際のガイドです。

## 詳細リファレンス

- [API リファレンス](references/api-reference.md) - `@xrift/world-components` の全フック・コンポーネント・定数の仕様
- [コードテンプレート](references/code-templates.md) - GLBモデル、テクスチャ、Skybox、インタラクション等の実装パターン
- [型定義](references/type-definitions.md) - User, PlayerMovement, VRTrackingData, TeleportDestination の型定義

## 最重要ルール（必ず守ること）

1. **アセット読み込みは必ず `useXRift()` の `baseUrl` を使用**
2. **アセットファイルは `public/` ディレクトリに配置**
3. **`baseUrl` は末尾に `/` を含むため、`${baseUrl}path` で結合**（`${baseUrl}/path` は NG）

```typescript
// 正しい
const { baseUrl } = useXRift()
const model = useGLTF(`${baseUrl}robot.glb`)

// 間違い
const model = useGLTF('/robot.glb')           // 絶対パス NG
const model = useGLTF(`${baseUrl}/robot.glb`) // 余分な / NG
```

## プロジェクト概要

- **用途**: XRiftプラットフォーム用WebXRワールド
- **技術**: React Three Fiber + Rapier物理エンジン + Module Federation
- **動作**: CDNにアップロード後、フロントエンドから動的ロード

## プロジェクト構造

```
xrift-world-template/
├── public/              # アセットファイル（直接配置、サブディレクトリ不要）
│   ├── model.glb
│   ├── texture.jpg
│   └── skybox.jpg
├── src/
│   ├── components/      # 3Dコンポーネント
│   ├── World.tsx        # メインワールドコンポーネント
│   ├── dev.tsx          # 開発用エントリーポイント
│   ├── index.tsx        # 本番用エクスポート
│   └── constants.ts     # 定数定義
├── .triplex/            # Triplex（3Dエディタ）設定
├── xrift.json           # XRift CLI設定
├── vite.config.ts       # ビルド設定（Module Federation）
└── package.json
```

## xrift.json 設定

### physics（物理演算設定）

| 項目 | 型 | デフォルト値 | 説明 |
|------|-----|-----------|------|
| `gravity` | number | 9.81 | 重力の強さ（正の値、地球=9.81、月=1.62、木星=24.79） |
| `allowInfiniteJump` | boolean | true | 無限ジャンプを許可するか |

```json
{
  "physics": {
    "gravity": 9.81,
    "allowInfiniteJump": true
  }
}
```

**設定例**:
- **アスレチックワールド**: `"allowInfiniteJump": false` で落下のリスクを追加
- **低重力ワールド**: `"gravity": 1.62`（月の重力）でふわふわした動き
- **高重力ワールド**: `"gravity": 24.79`（木星の重力）で重厚な動き

## コマンドリファレンス

```bash
# 開発
npm run dev        # 開発サーバー起動 (http://localhost:5173)
npm run build      # 本番ビルド
npm run typecheck  # 型チェック

# XRift CLI
xrift login        # 認証
xrift create       # 新規プロジェクト作成
xrift upload world # ワールドをアップロード
xrift whoami       # ログインユーザー確認
xrift logout       # ログアウト
```

## 開発環境での動作確認

`npm run dev` で開発サーバーを起動すると、一人称視点でワールドを操作・テストできる。

| 操作 | キー |
|------|------|
| 視点操作 | 画面クリックでマウスロック → マウス移動 |
| 移動 | W / A / S / D |
| 上昇 / 下降 | E・Space / Q |
| インタラクト | 照準を合わせてクリック |
| マウスロック解除 | ESC |

`Interactable` コンポーネントのクリック動作も開発環境で確認可能（画面中央の Raycaster が `LAYERS.INTERACTABLE` レイヤーを検出）。

### dev.tsx の構成

`src/dev.tsx` は開発専用のエントリーポイント。本番ビルドには含まれない。

**注意**: 本番環境では `XRiftProvider` は不要（フロントエンド側が自動でラップ）

## 依存パッケージ

### 必須（peerDependencies）
- `react` / `react-dom` ^19.0.0
- `three` ^0.182.0
- `@react-three/fiber` ^9.3.0
- `@react-three/drei` ^10.7.3
- `@react-three/rapier` ^2.1.0

### XRift固有
- `@xrift/world-components` - XRiftのフック・コンポーネント

## トラブルシューティング

### "useXRift must be used within XRiftProvider"

**原因**: `XRiftProvider` でラップされていない

**解決方法**:
- `src/dev.tsx` で `XRiftProvider` を使用しているか確認
- Triplex使用時: `.triplex/provider.tsx` を確認

### アセットが読み込めない

**原因**: `baseUrl` を使用していない、またはパス結合が間違っている

**解決方法**:
```typescript
// 正しい
const { baseUrl } = useXRift()
const model = useGLTF(`${baseUrl}robot.glb`)

// 間違い
const model = useGLTF('/robot.glb')
const model = useGLTF(`${baseUrl}/robot.glb`)
```

### 物理演算が効かない

**原因**: `Physics` コンポーネントでラップされていない、または `RigidBody` がない

**解決方法**:
```typescript
<Physics>
  <RigidBody type="fixed">  {/* または "dynamic" */}
    <mesh>...</mesh>
  </RigidBody>
</Physics>
```

## 参考リンク

- [XRift ドキュメント](https://docs.xrift.net)
- [XRift CLI (GitHub)](https://github.com/WebXR-JP/xrift-cli)
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber)
- [Rapier Physics](https://rapier.rs/docs/)
- [Triplex（ビジュアルエディタ）](https://triplex.dev/)

## 実装例の参照先

- **GLBモデル**: `src/components/Duck/index.tsx`
- **Skybox**: `src/components/Skybox/index.tsx`
- **アニメーション**: `src/components/RotatingObject/index.tsx`
- **インタラクション**: `src/components/InteractableButton/index.tsx`
- **ユーザー追跡**: `src/components/RemoteUserHUDs/index.tsx`
- **テレポート**: `src/components/TeleportPortal/index.tsx`
- **メインワールド**: `src/World.tsx`
