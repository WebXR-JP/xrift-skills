# XRift Skills

[English](README.en.md)

XRift で開発するための [Agent Skills](https://github.com/vercel-labs/skills)。
AI コーディングエージェント（Claude Code, Cursor, Copilot, Codex 等）が、XRift のワールド制作、ワールドやアイテムのアップロード、Public API を使ったデータ取得を行う際に必要な情報を提供します。

## インストール

```bash
npx skills add WebXR-JP/xrift-skills
```

## 含まれるスキル

### xrift-world

XRift プラットフォーム用 WebXR ワールド制作ガイド。

- **SKILL.md** - 最重要ルール、対応端末（PC・モバイル・VR）のルール、プロジェクト概要、設定、コマンド、トラブルシューティング
- **references/api-reference.md** - `@xrift/world-components` のフック・コンポーネント・定数の全仕様
- **references/code-templates.md** - GLB モデル、テクスチャ、Skybox、インタラクション等のコードテンプレート
- **references/type-definitions.md** - User, PlayerMovement, VRTrackingData 等の型定義

### xrift-sdk

`@xrift/sdk` でワールド・アイテムをプログラムからアップロードするためのガイド。

- **SKILL.md** - 最重要ルール、XriftClient の初期化、アップロードの流れ、エラー処理
- **references/api-reference.md** - XriftClient, WorldsApi, ItemsApi, エラークラス, ユーティリティの全仕様
- **references/code-templates.md** - Node.js・ブラウザ環境それぞれのコード例
- **references/type-definitions.md** - SDK の全インターフェースの型定義

### xrift-public-api

API キーを使って Public API v1（REST）から XRift のデータを読むためのガイド。ボット・ダッシュボード・外部ツールの開発向け。

- **SKILL.md** - 最重要ルール、認証、スコープ、ページ指定の規則、レート制限、マーケットプレイス・イベント・フレンドの考え方
- **references/api-reference.md** - ワールド・ユーザー・インスタンス・マーケットプレイス・イベント・フレンド・統計の全エンドポイントの仕様
- **references/code-templates.md** - fetch を使った最小のクライアント、全ページの取得、イベントの取得、オンラインのフレンドの取得、エラー処理のコード例

## 更新

インストール済みのスキルを最新版に更新するには：

```bash
npx skills update
```

## 関連リンク

- [XRift ドキュメント](https://docs.xrift.net)
- [xrift-world-template](https://github.com/WebXR-JP/xrift-world-template)
- [XRift CLI](https://github.com/WebXR-JP/xrift-cli)
- [Agent Skills Directory](https://skills.sh)

## ライセンス

MIT
