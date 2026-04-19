# XRift Skills

[English](README.en.md)

XRift ワールド制作のための [Agent Skills](https://github.com/vercel-labs/skills)。
AI コーディングエージェント（Claude Code, Cursor, Copilot, Codex 等）で XRift ワールドを作成する際に必要な情報を提供します。

## インストール

### 最短導入（推奨）

全プラットフォーム共通。スキルだけをシンプルに入れたい場合はこちら：

```bash
npx skills add WebXR-JP/xrift-skills
```

### プラグイン経由で導入（マーケットプレイス / slash command 対応）

各プラットフォームの plugin システム経由で導入すると、マーケットプレイス統合や slash command、自動更新などが使えます。

#### Claude Code

マーケットプレイスを登録：

```
/plugin marketplace add WebXR-JP/xrift-skills
```

プラグインをインストール：

```
/plugin install xrift-skills@xrift-marketplace
```

#### OpenAI Codex

Codex のマーケットプレイスに登録（Codex CLI 0.121 以降）：

```bash
codex marketplace add https://github.com/WebXR-JP/xrift-skills
```

Codex セッションで `/plugins` を実行し、一覧から `xrift-skills` を選んで `Install Plugin`。

旧バージョン向けの手動導入は [`.codex/INSTALL.md`](.codex/INSTALL.md) を参照。

#### Cursor

Cursor Agent チャットで以下を実行（Cursor 2.5 以降）：

```
/add-plugin xrift-skills@https://github.com/WebXR-JP/xrift-skills
```

#### OpenCode

OpenCode に以下のように指示：

```
Fetch and follow instructions from https://raw.githubusercontent.com/WebXR-JP/xrift-skills/main/.opencode/INSTALL.md
```

詳細手順：[`.opencode/INSTALL.md`](.opencode/INSTALL.md)

#### GitHub Copilot CLI

```bash
copilot plugin marketplace add WebXR-JP/xrift-skills
copilot plugin install xrift-skills@xrift-marketplace
```

#### Gemini CLI

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

最短導入の場合:

```bash
npx skills update
```

Claude Code（plugin 経由）:

```
/plugin marketplace update xrift-marketplace
```

Gemini CLI（plugin 経由）:

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
