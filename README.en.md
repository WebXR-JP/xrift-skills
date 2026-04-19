# XRift Skills

[日本語](README.md)

[Agent Skills](https://github.com/vercel-labs/skills) for building XRift worlds.
Provides essential information for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) when creating XRift worlds.

## Installation

**Note:** Installation differs by platform.

### Claude Code

Register the marketplace:

```
/plugin marketplace add WebXR-JP/xrift-skills
```

Install the plugin:

```
/plugin install xrift-skills@xrift-marketplace
```

### OpenAI Codex

Codex uses native skill discovery. Follow:

[`.codex/INSTALL.md`](.codex/INSTALL.md)

### Cursor

Load `.cursor-plugin/plugin.json` via Cursor's plugin marketplace or plugin settings.

### OpenCode

Tell OpenCode:

```
Fetch and follow instructions from https://raw.githubusercontent.com/WebXR-JP/xrift-skills/main/.opencode/INSTALL.md
```

Detailed instructions: [`.opencode/INSTALL.md`](.opencode/INSTALL.md)

### GitHub Copilot CLI

```bash
copilot plugin marketplace add WebXR-JP/xrift-skills
copilot plugin install xrift-skills@xrift-marketplace
```

### Gemini CLI

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

Claude Code:

```
/plugin marketplace update xrift-marketplace
```

Gemini CLI:

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
