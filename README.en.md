# XRift Skills

[日本語](README.md)

[Agent Skills](https://github.com/vercel-labs/skills) for developing with XRift.
Provides essential information for AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) when building XRift worlds, uploading worlds and items, and fetching data from the Public API.

## Installation

```bash
npx skills add WebXR-JP/xrift-skills
```

## Included Skills

### xrift-world

A guide for building WebXR worlds on the XRift platform.

- **SKILL.md** - Critical rules, device support rules (PC, mobile, VR), project overview, configuration, commands, troubleshooting
- **references/api-reference.md** - Full specification of `@xrift/world-components` hooks, components, and constants
- **references/code-templates.md** - Code templates for GLB models, textures, Skybox, interactions, and more
- **references/type-definitions.md** - Type definitions for User, PlayerMovement, VRTrackingData, etc.

### xrift-sdk

A guide for programmatically uploading worlds and items with `@xrift/sdk`.

- **SKILL.md** - Critical rules, XriftClient initialization, upload flows, error handling
- **references/api-reference.md** - Full specification of XriftClient, WorldsApi, ItemsApi, error classes, and utility functions
- **references/code-templates.md** - Code examples for Node.js and browser environments
- **references/type-definitions.md** - Complete type definitions for all SDK interfaces

### xrift-public-api

A guide for reading XRift data from the Public API v1 (REST) with an API key. For building bots, dashboards, and external tools.

- **SKILL.md** - Critical rules, authentication, scopes, pagination rules, rate limits, marketplace, event, and friend concepts
- **references/api-reference.md** - Specification of every endpoint: worlds, users, instances, marketplace, events, friends, and stats
- **references/code-templates.md** - `fetch` examples: a minimal client, fetching all pages, events, online friends, error handling

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
