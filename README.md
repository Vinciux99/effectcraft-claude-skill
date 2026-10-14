# EffectCraft skill for Claude

EffectCraft is an After Effects-style motion graphics and compositing app from the open-source ArtCraft suite (storytold/effectcraft, project files `.ecproj`). It is young software (the version this skill was written against is 0.9.x), so expect gaps.

This repository packages a **skill** that teaches Claude to drive EffectCraft (part of the [ArtCraft suite](https://github.com/storytold/effectcraft) by storytold) and that **knows how to create the MCP server and guide you through it**, on **Windows and macOS**. It is unofficial and not affiliated with storytold.

> Part of the [ArtCraft Claude skills hub](https://github.com/Vinciux99/artcraft-claude-skills): all 13 skills, a picker and a one-command installer.

## What it does
- Operating manual for EffectCraft: connection modes, ground rules, every MCP tool, recipes, CLI, control channel, command families and a safe workflow (work on copies, verify, save only with your OK).
- Built-in MCP setup assistant: Claude checks whether the server is connected, then walks you through installing it (automatic script or manual config), the launcher, authentication and verification.
- Troubleshooting table for the usual problems (server "disconnected", ports, macOS quarantine, Windows Store config location...).

## What you can ask Claude
- "Create a 1080p 30 fps lower-third with my name sliding in and a soft glow."
- "Animate this logo layer: scale from 0 to 100 with easy ease over half a second, then add a subtle wiggle."
- "Build a 10-second kinetic-typography intro from these lines, using Inter SemiBold."
- "Apply a Gaussian Blur to layer 3 and keyframe the blurriness from 40 to 0."
- "Render frame 1.5 s at 640 px so I can check the timing."
- "Export this comp as ProRes 4444 with alpha."
- "Write an After Effects-style script that makes 20 staggered circles and run it."
- "Export the comp to Lottie and tell me which features produced warnings."

## Highlights
- Three interfaces explained (MCP, CLI, control channel) and when to pick each.
- Full MCP tool table, reference syntax (layer/comp/property paths) and time-base gotchas.
- Recipes for keyframes, effects, masks, expressions, text, rendering, Lottie, and Premiere interchange.
- AE-style scripting cheat sheet and the list of engine commands by menu.
- Built-in guidance for creating and troubleshooting the MCP server on Windows and macOS.

## App-specific notes
- Kit port **9877** (same as upstream docs). Live MCP uses `mcp --bridge 9877`; headless uses `mcp --autosave`.
- The control channel has no authentication and listens on loopback only.
- Extra fonts: set `EFFECTCRAFT_FONT_DIRS`.

## Get the skill

| Where | How |
|---|---|
| **Claude app** (claude.ai / Desktop) | Download [`effectcraft-skill.zip](https://github.com/Vinciux99/effectcraft-claude-skill/releases/latest/download/effectcraft-skill.zip)`, then Customize > Skills > "+" > Create skill > Upload a skill. Needs Settings > Capabilities > "Code execution and file creation" on. |
| **Claude Code, plugin** | `/plugin marketplace add Vinciux99/artcraft-claude-skills` then `/plugin install effectcraft@artcraft` |
| **Claude Code, manual** | copy `skills/effectcraft/` to `~/.claude/skills/effectcraft/` (the setup script below does it) |

## Create the MCP server

Prerequisite: install EffectCraft from [upstream releases](https://github.com/storytold/effectcraft/releases) (Windows `.msi`, macOS `.dmg`) and Claude Desktop and/or Claude Code.

**One command (recommended)**

Windows (PowerShell):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((irm https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.ps1))) -Apps effectcraft"
```
macOS (Terminal):
```sh
curl -fsSL https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.sh | bash -s -- --apps effectcraft
```
Add `-DryRun` (Windows) or `--dry-run` (macOS) to preview, and `-Mode live|headless|both` / `--mode ...` to choose the mode.

**Or from a clone:** double-click `setup-windows.cmd` (Windows) or `setup-macos.command` (macOS; first time right-click > Open).

**Or just ask Claude:** once the skill is installed, say "set up the EffectCraft MCP server" and Claude will guide you step by step.

After setup: fully quit and reopen Claude, open EffectCraft with the **"EffectCraft (Claude)"** launcher, then ask Claude to use it. If the server shows "disconnected", the app was not running when Claude started: open it with the launcher and re-enable the server, or use headless mode.

What the setup changes (all backed up or reversible): Claude Desktop config (`mcpServers.effectcraft`, backup `<file>.backup-<timestamp>`), the launcher, Claude Code's user-scope MCP list and `~/.claude/skills/effectcraft/`.

## Repository layout
```
skills/effectcraft/SKILL.md      the skill
app.json                   machine-readable settings (ports, paths, MCP args) used by the scripts
scripts/                   artcraft-setup.ps1 (Windows), artcraft-setup.sh (macOS), json-tool.js
setup-windows.cmd          double-click wrapper
setup-macos.command        double-click wrapper
.claude-plugin/plugin.json Claude Code plugin manifest
.github/workflows/ci.yml   validation, setup tests on Windows and macOS, release zip
```

## CI and releases
Every push validates `app.json`, `SKILL.md` and the scripts, then runs the setup scripts against a fake install on **Windows** and **macOS** runners. Pushing a tag `v*` builds `effectcraft-skill.zip` and publishes a GitHub release.

## Status and caveats
- Windows paths and arguments come from a working install. The macOS paths (`/Applications/EffectCraft.app`, CLI location and release asset name) follow the upstream packaging but have **not** been verified on a Mac; the skill marks unverified items and the setup script reports what it could not find.
- EffectCraft is a young open-source project; behaviour can change between releases. `list_commands` / `effectcraft-cli commands` is always the source of truth.
- Unofficial community packaging, MIT licensed (see `LICENSE`).
