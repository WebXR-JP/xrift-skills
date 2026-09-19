# XRift Skills

[日本語](README.md)

[Agent Skills](https://github.com/vercel-labs/skills) for building XRift worlds.
Provides essential information for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) when creating XRift worlds.

## Installation

```bash
npx skills add WebXR-JP/xrift-skills
```

## Included Skills

### xrift-world

A guide for building WebXR worlds on the XRift platform.

- **SKILL.md** - Critical rules, project overview, configuration, commands, troubleshooting
- **references/api-reference.md** - Full specification of `@xrift/world-components` hooks, components, and constants
- **references/code-templates.md** - Code templates for GLB models, textures, Skybox, interactions, and more
- **references/type-definitions.md** - Type definitions for User, PlayerMovement, VRTrackingData, etc.

### xrift-sdk

A guide for programmatically uploading worlds and items with `@xrift/sdk`.

- **SKILL.md** - Critical rules, XriftClient initialization, upload flows, error handling
- **references/api-reference.md** - Full specification of XriftClient, WorldsApi, ItemsApi, error classes, and utility functions
- **references/code-templates.md** - Code examples for Node.js and browser environments
- **references/type-definitions.md** - Complete type definitions for all SDK interfaces

### xrift-world-editing

A guide for editing XRift worlds from the browser through WebMCP. While the user is inside an
instance on `app.xrift.net`, an AI agent calls the tools the page exposes.

- **SKILL.md** - Where the tools exist, coordinate and rotation conventions, limits, and the rules that keep the agent out of the user's way
- **references/tool-reference.md** - Input/output schema for all eight tools, placeable types, and error messages

## Updating

To update installed skills to the latest version:

```bash
npx skills update
```

## Links

- [XRift Documentation](https://docs.xrift.net)
- [xrift-world-template](https://github.com/WebXR-JP/xrift-world-template)
- [XRift CLI](https://github.com/WebXR-JP/xrift-cli)
- [Agent Skills Directory](https://skills.sh)

## License

MIT
