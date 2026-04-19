# XRift Skills

[日本語](README.md)

[Agent Skills](https://github.com/vercel-labs/skills) for building XRift worlds.
Provides essential information for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) when creating XRift worlds.

## Installation

### Quick install (recommended)

Cross-platform one-liner for installing just the skills:

```bash
npx skills add WebXR-JP/xrift-skills
```

### Plugin install (marketplace / slash command support)

For marketplace integration, slash commands, and auto-updates, install via each platform's plugin system:

#### Claude Code

Register the marketplace:

```
/plugin marketplace add WebXR-JP/xrift-skills
```

Install the plugin:

```
/plugin install xrift-skills@xrift-marketplace
```

#### OpenAI Codex

Add this marketplace (Codex CLI 0.121+):

```bash
codex marketplace add https://github.com/WebXR-JP/xrift-skills
```

Then run `/plugins` in a Codex session and select `xrift-skills` → `Install Plugin`.

Manual install for older Codex versions: [`.codex/INSTALL.md`](.codex/INSTALL.md).

#### Cursor

In Cursor Agent chat, run (Cursor 2.5+):

```
/add-plugin xrift-skills@https://github.com/WebXR-JP/xrift-skills
```

#### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/WebXR-JP/xrift-skills/main/.opencode/INSTALL.md
```

Detailed instructions: [`.opencode/INSTALL.md`](.opencode/INSTALL.md)

#### GitHub Copilot CLI

```bash
copilot plugin marketplace add WebXR-JP/xrift-skills
copilot plugin install xrift-skills@xrift-marketplace
```

#### Gemini CLI

```bash
gemini extensions install https://github.com/WebXR-JP/xrift-skills
```

## Included Skills

### xrift-world

A guide for building WebXR worlds on the XRift platform.

- **SKILL.md** - Critical rules, project overview, configuration, commands, troubleshooting
- **references/api-reference.md** - Full specification of `@xrift/world-components` hooks, components, and constants
- **references/code-templates.md** - Code templates for GLB models, textures, Skybox, interactions, and more
- **references/type-definitions.md** - Type definitions for User, PlayerMovement, VRTrackingData, etc.

## Updating

Quick install:

```bash
npx skills update
```

Claude Code (plugin):

```
/plugin marketplace update xrift-marketplace
```

Gemini CLI (plugin):

```bash
gemini extensions update xrift-skills
```

## Links

- [XRift Documentation](https://docs.xrift.net)
- [xrift-world-template](https://github.com/WebXR-JP/xrift-world-template)
- [XRift CLI](https://github.com/WebXR-JP/xrift-cli)
- [Agent Skills Directory](https://skills.sh)

## License

MIT
