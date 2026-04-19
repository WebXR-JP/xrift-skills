# XRift Skills

[English](README.en.md)

XRift ワールド制作のための [Agent Skills](https://github.com/vercel-labs/skills)。
AI コーディングエージェント（Claude Code, Cursor, Copilot, Codex 等）で XRift ワールドを作成する際に必要な情報を提供します。

## インストール

**注意:** インストール方法はプラットフォームごとに異なります。

### Claude Code

マーケットプレイスを登録：

```
/plugin marketplace add WebXR-JP/xrift-skills
```

プラグインをインストール：

```
/plugin install xrift-skills@xrift-marketplace
```

### OpenAI Codex

Codex はネイティブの skill 探索機能を使用します。手順：

[`.codex/INSTALL.md`](.codex/INSTALL.md)

### Cursor

Cursor の plugin マーケットプレイス、またはプラグイン設定から `.cursor-plugin/plugin.json` を読み込んでください。

### OpenCode

OpenCode に以下のように指示：

```
Fetch and follow instructions from https://raw.githubusercontent.com/WebXR-JP/xrift-skills/main/.opencode/INSTALL.md
```

詳細手順：[`.opencode/INSTALL.md`](.opencode/INSTALL.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add WebXR-JP/xrift-skills
copilot plugin install xrift-skills@xrift-marketplace
```

### Gemini CLI

```bash
gemini extensions install https://github.com/WebXR-JP/xrift-skills
```

## 含まれるスキル

### xrift-world

XRift プラットフォーム用 WebXR ワールド制作ガイド。

- **SKILL.md** - 最重要ルール、プロジェクト概要、設定、コマンド、トラブルシューティング
- **references/api-reference.md** - `@xrift/world-components` のフック・コンポーネント・定数の全仕様
- **references/code-templates.md** - GLB モデル、テクスチャ、Skybox、インタラクション等のコードテンプレート
- **references/type-definitions.md** - User, PlayerMovement, VRTrackingData 等の型定義

## 更新

Claude Code:

```
/plugin marketplace update xrift-marketplace
```

Gemini CLI:

```bash
gemini extensions update xrift-skills
```

## 関連リンク

- [XRift ドキュメント](https://docs.xrift.net)
- [xrift-world-template](https://github.com/WebXR-JP/xrift-world-template)
- [XRift CLI](https://github.com/WebXR-JP/xrift-cli)
- [Agent Skills Directory](https://skills.sh)

## ライセンス

MIT
