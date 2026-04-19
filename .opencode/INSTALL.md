# Installing xrift-skills for OpenCode

OpenCode loads skills from `~/.opencode/skills/`. Install by cloning the repo and symlinking its `skills/` directory.

## 1. Clone

```bash
git clone https://github.com/WebXR-JP/xrift-skills.git ~/.opencode-src/xrift-skills
```

## 2. Symlink

**macOS / Linux:**

```bash
mkdir -p ~/.opencode/skills
ln -s ~/.opencode-src/xrift-skills/skills ~/.opencode/skills/xrift-skills
```

**Windows (PowerShell, run as Administrator OR with Developer Mode enabled):**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.opencode\skills" | Out-Null
cmd /c mklink /J "$env:USERPROFILE\.opencode\skills\xrift-skills" "$env:USERPROFILE\.opencode-src\xrift-skills\skills"
```

## 3. Restart OpenCode

Open a new OpenCode session and confirm the `xrift-world` skill is discoverable.
