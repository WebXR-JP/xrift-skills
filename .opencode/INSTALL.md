# Installing xrift-skills for OpenCode

OpenCode auto-discovers Agent Skills from `~/.config/opencode/skills/<name>/SKILL.md` (global) or `.opencode/skills/<name>/SKILL.md` (per-project). Clone this repo and symlink the `xrift-world` skill directory.

## 1. Clone

```bash
git clone https://github.com/WebXR-JP/xrift-skills.git ~/src/xrift-skills
```

(Clone anywhere you like; adjust the symlink source path below.)

## 2. Symlink

**macOS / Linux:**

```bash
mkdir -p ~/.config/opencode/skills
ln -s ~/src/xrift-skills/skills/xrift-world ~/.config/opencode/skills/xrift-world
```

**Windows (PowerShell, run as Administrator OR with Developer Mode enabled):**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.config\opencode\skills" | Out-Null
cmd /c mklink /J "$env:USERPROFILE\.config\opencode\skills\xrift-world" "$env:USERPROFILE\src\xrift-skills\skills\xrift-world"
```

## 3. Restart OpenCode

Open a new OpenCode session and confirm the `xrift-world` skill is discoverable.
