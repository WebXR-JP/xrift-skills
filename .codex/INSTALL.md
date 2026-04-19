# Installing xrift-skills for Codex

Codex discovers skills natively from `~/.agents/skills/`. Install by cloning the repo and symlinking its `skills/` directory.

## 1. Clone

```bash
git clone https://github.com/WebXR-JP/xrift-skills.git ~/.codex/xrift-skills
```

## 2. Symlink

**macOS / Linux:**

```bash
mkdir -p ~/.agents/skills
ln -s ~/.codex/xrift-skills/skills ~/.agents/skills/xrift-skills
```

**Windows (PowerShell, run as Administrator OR with Developer Mode enabled):**

```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.agents\skills" | Out-Null
cmd /c mklink /J "$env:USERPROFILE\.agents\skills\xrift-skills" "$env:USERPROFILE\.codex\xrift-skills\skills"
```

## 3. Restart Codex

Open a new Codex session. Ask "what skills do you have?" — `xrift-world` should appear.
