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
