# XRift Skills

[English](README.en.md)

XRift ワールド制作のための [Agent Skills](https://github.com/vercel-labs/skills)。
AI コーディングエージェント（Claude Code, Cursor, Copilot, Codex 等）で XRift ワールドを作成する際に必要な情報を提供します。

## インストール

```bash
npx skills add WebXR-JP/xrift-skills
```

## 含まれるスキル

### xrift-world

XRift プラットフォーム用 WebXR ワールド制作ガイド。

- **SKILL.md** - 最重要ルール、プロジェクト概要、設定、コマンド、トラブルシューティング
- **references/api-reference.md** - `@xrift/world-components` のフック・コンポーネント・定数の全仕様
- **references/code-templates.md** - GLB モデル、テクスチャ、Skybox、インタラクション等のコードテンプレート
- **references/type-definitions.md** - User, PlayerMovement, VRTrackingData 等の型定義

### xrift-sdk

`@xrift/sdk` でワールド・アイテムをプログラムからアップロードするためのガイド。

- **SKILL.md** - 最重要ルール、XriftClient の初期化、アップロードの流れ、エラー処理
- **references/api-reference.md** - XriftClient, WorldsApi, ItemsApi, エラークラス, ユーティリティの全仕様
- **references/code-templates.md** - Node.js・ブラウザ環境それぞれのコード例
- **references/type-definitions.md** - SDK の全インターフェースの型定義

### xrift-world-editing

ブラウザから WebMCP 経由で XRift のワールドを編集するためのガイド。
`app.xrift.net` でインスタンスに入っているあいだ、ページが公開するツールを AI が呼ぶ。

- **SKILL.md** - ツールが生える条件、座標・回転の規約、上限、ユーザーの操作を邪魔しないための決まり
- **references/tool-reference.md** - 8本のツールの入出力スキーマ、置ける種別、エラーメッセージ

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
