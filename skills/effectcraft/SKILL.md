---
name: effectcraft
description: Create and edit motion graphics and compositions with EffectCraft (After Effects-style, ArtCraft suite) through MCP, the CLI or the control channel, and set up, connect or troubleshoot its MCP server on Windows and macOS.
---

# EffectCraft for Claude

EffectCraft is an After Effects-style motion graphics and compositing app from the open-source ArtCraft suite (storytold/effectcraft, project files `.ecproj`). It is young software (the version this skill was written against is 0.9.x), so expect gaps.

**Everything is an engine command** with a stable id and JSON parameters (`comp.new`, `layer.newText`, `effect.apply`, ...). Menus, shortcuts, panel gestures, the CLI, the control channel and MCP all go through the same commands, so an agent can do anything a user can, and every change is undoable.

## Quick check: is the MCP server connected?

1. Look at the tools you have. If tools from the MCP server `effectcraft` are available (for example `list_commands`, `execute` or `inspect_*` tools) the server is connected: skip to "Operating knowledge" below and use them directly.
2. If they are missing, check whether the user installed it under the name `effectcraft` or `effectcraft-headless`. In Claude Desktop the server may show as **disconnected**.
3. **Most common cause of "disconnected":** the live server (`mcp --bridge {port}`) exits when EffectCraft is not running at the moment Claude starts. Fix: open EffectCraft with the **"EffectCraft (Claude)" launcher** (it starts the app with its control channel on), then re-enable the server in Claude's settings or fully quit and reopen Claude.
4. If the user does not need to watch the app live, **headless mode** always starts (no window, works on files). See "Live, headless or both".
5. If nothing is configured yet, follow "Set up the MCP server" below. Ask the user which OS they use if you do not know.

## Set up the MCP server (guide the user)

Walk the user through this step by step. Ask for the OS (Windows or macOS) first. If you have a shell on the user's machine you may run the steps yourself, but **ask before editing any config file and always back it up first**. Never print tokens or keys in chat.

### 1. Prerequisites
- Install EffectCraft from its upstream releases page: https://github.com/storytold/effectcraft/releases (Windows: the `.msi` installer; macOS: the `.dmg`, then drag the app to `/Applications`).
- Install Claude Desktop and/or Claude Code. For Claude Desktop, fully quit and reopen it after any config change.
- The MCP server is the **CLI** that ships with the app: `effectcraft-cli`. Windows installs it next to the app (`C:\Program Files\EffectCraft\effectcraft-cli.exe`, not on `PATH`). On macOS the CLI is a separate release download (asset `effectcraft-cli-<version>-macos-universal.zip`) unless it is bundled in the app (unverified): the setup script below fetches it into `~/.artcraft/bin/` and clears the macOS quarantine flag.
- This kit uses port **9877** (the upstream default), which does not clash with the other apps of the kit.

### 2. Option A: automatic setup (recommended)
The setup script finds the app and CLI, creates the launcher, backs up and updates Claude Desktop's config, registers the server in Claude Code (if the `claude` command exists) and installs this skill. Preview first with a dry run: nothing is changed.

Windows (PowerShell):
```powershell
powershell -NoProfile -ExecutionPolicy Bypass -Command "& ([scriptblock]::Create((irm https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.ps1))) -Apps effectcraft"
```
macOS (Terminal):
```sh
curl -fsSL https://raw.githubusercontent.com/Vinciux99/artcraft-claude-skills/main/install.sh | bash -s -- --apps effectcraft
```
Or clone/download this repository and double-click `setup-windows.cmd` (Windows) or `setup-macos.command` (macOS; first time: right-click, Open). Useful options: `-Mode live|headless|both` / `--mode ...` (default `live`; `both` registers `effectcraft` live and `effectcraft-headless`), `-DryRun` / `--dry-run`, `-NoLauncher`, `-NoClaudeDesktop`, `-NoClaudeCode`, `-NoSkill` (macOS: `--no-launcher`, `--no-claude-desktop`, `--no-claude-code`, `--no-skill`).

What it changes: Claude Desktop config (`%APPDATA%\Claude\claude_desktop_config.json` plus the Microsoft Store/MSIX copy under `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`; macOS `~/Library/Application Support/Claude/claude_desktop_config.json`; backup saved next to it as `<file>.backup-<timestamp>`), the launcher "EffectCraft (Claude)" (Windows: Desktop shortcut; macOS: `~/Applications/EffectCraft (Claude).app` plus a Desktop alias), Claude Code's user-scope MCP list, and `~/.claude/skills/effectcraft/`.

### 3. Option B: manual setup
**Windows** - edit `%APPDATA%\Claude\claude_desktop_config.json` (if Claude was installed from the Microsoft Store, also the copy under `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\`). Keep other servers; add this inside `mcpServers`. Backslashes must be doubled in JSON.
```json
{
  "mcpServers": {
    "effectcraft": {
      "command": "C:\\Program Files\\EffectCraft\\effectcraft-cli.exe",
      "args": [
        "mcp",
        "--bridge",
        "9877"
      ]
    },
    "effectcraft-headless": {
      "command": "C:\\Program Files\\EffectCraft\\effectcraft-cli.exe",
      "args": [
        "mcp",
        "--autosave"
      ]
    }
  }
}
```
**macOS** - edit `~/Library/Application Support/Claude/claude_desktop_config.json`. Use the absolute CLI path (replace `YOUR_NAME`; JSON does not expand `~`). The CLI path is unverified if your install differs: check with `ls ~/.artcraft/bin /Applications/EffectCraft.app/Contents/MacOS`.
```json
{
  "mcpServers": {
    "effectcraft": {
      "command": "/Users/YOUR_NAME/.artcraft/bin/effectcraft-cli",
      "args": [
        "mcp",
        "--bridge",
        "9877"
      ]
    },
    "effectcraft-headless": {
      "command": "/Users/YOUR_NAME/.artcraft/bin/effectcraft-cli",
      "args": [
        "mcp",
        "--autosave"
      ]
    }
  }
}
```
**Launcher** (starts the app with the control channel; needed before the live server can connect):
- Windows: create a Desktop shortcut named "EffectCraft (Claude)" with target `"C:\Program Files\EffectCraft\effectcraft.exe" --control 9877`
- macOS: run `open -n -a "/Applications/EffectCraft.app" --args --control 9877`, or save it as an app with Script Editor/`osacompile` (the setup script does this for you).

**Claude Code** (any OS; run once):
Windows (PowerShell):
```powershell
claude mcp add --scope user effectcraft -- "C:\Program Files\EffectCraft\effectcraft-cli.exe" mcp --bridge 9877
claude mcp add --scope user effectcraft-headless -- "C:\Program Files\EffectCraft\effectcraft-cli.exe" mcp --autosave
```
macOS (Terminal):
```sh
claude mcp add --scope user effectcraft -- "$HOME/.artcraft/bin/effectcraft-cli" mcp --bridge 9877
claude mcp add --scope user effectcraft-headless -- "$HOME/.artcraft/bin/effectcraft-cli" mcp --autosave
```
Check with `claude mcp list`.

### 4. Authentication
EffectCraft needs no token: the control channel listens on loopback (`127.0.0.1`) only and has no authentication. Anyone on the machine can drive the app while it is open, so close it when you are done.

### 5. Finish and verify
1. **Fully quit and reopen Claude** (Desktop: quit from the tray/menu bar, not just close the window).
2. Open EffectCraft with the **"EffectCraft (Claude)" launcher** before asking Claude to use it.
3. In Claude, confirm the server `effectcraft` is connected and call its `list_commands` tool (or equivalent). A non-empty command list means it works.

### Live, headless or both
- **Live** (`effectcraft`): drives the open app; the user sees every change and screenshots/UI tools work. Needs the launcher first.
- **Headless** (`effectcraft-headless`): in-process engine, no window; always starts; works on files; UI/screenshot tools are not available.
- **Both**: register both and pick per task. Switch with `-Mode` / `--mode`, or by editing the config entries above.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Server shows "disconnected" right after Claude starts | Live server exits when EffectCraft is not running | Open EffectCraft with the "EffectCraft (Claude)" launcher, then re-enable the server or restart Claude; or use headless mode |
| "Port already in use" / app opens but Claude cannot connect | Another app or an old EffectCraft instance holds port 9877 | Quit all EffectCraft windows, check the launcher uses the kit port, make sure no other ArtCraft app uses the same port |
| `command not found` / spawn error | CLI path wrong or the CLI is not installed (Windows: not on `PATH`; macOS: separate download) | Use the full absolute path in the config; macOS: run the setup script (it downloads the CLI to `~/.artcraft/bin/`) |
| macOS: "cannot be opened because the developer cannot be verified" | Gatekeeper/quarantine on the app or CLI | Right-click the app, Open; for the CLI: `xattr -dr com.apple.quarantine ~/.artcraft/bin/effectcraft-cli` |
| Anyone else can drive the app | The control channel has no authentication (loopback only) | Start the app with the control channel only when needed and quit it when done |
| Config edited but nothing changed (Windows) | Claude installed from the Microsoft Store reads the MSIX copy of the config | Edit/update `%LOCALAPPDATA%\Packages\Claude_*\LocalCache\Roaming\Claude\claude_desktop_config.json` too |
| JSON config error / server not listed | Unescaped backslashes or trailing comma | Double the backslashes on Windows; validate the JSON; restore the `.backup-*` file if needed |
| App crashes on start or window is blank | Graphics backend problem | Windows: set `WGPU_BACKEND=vulkan` (or `dx12`) for one launch; avoid `gl` on some AMD drivers |
| Changes do not appear in the window | You are on the headless server | Use the live server (launcher first) |
| Quotes disappear in PowerShell JSON arguments | Windows PowerShell 5.1 quoting for `.exe` | Use `--script`/files where the CLI supports them, or single-quote the whole JSON |

## Operating knowledge

## Three interfaces

| Interface | When to use it | Needs the window? |
|---|---|---|
| **MCP** `effectcraft-cli mcp` | Claude and other MCP clients | No (headless) or yes (`--bridge 9877`) |
| **CLI** `effectcraft-cli exec/get/set/...` | One-off calls and scripts; JSON output with `--json` | No |
| **Control channel** `effectcraft --control 9877` | Drive and *see* the live interface | Yes |

### Choosing the channel
1. **MCP in bridge (live) mode — preferred.** If a server named `effectcraft` is among your tools, use it. It needs the app started with `--control 9877` (or env `EFFECTCRAFT_CONTROL_PORT=9877`; the flag wins over the variable). The "EffectCraft (Claude)" launcher does that. Edits show up in the open window in real time.
2. **MCP headless.** `mcp` plus `--demo` or `--project file.ecproj`; `--gpu` for the GPU compositor; `--autosave` writes a checkpoint after each change (into `<config>/EffectCraft Auto-Save/MCP/`). Without autosave, work lives only in memory: call `save_project {path}` at every milestone.
3. **A shell on the user's machine** (Claude Code, desktop shell): use the CLI directly, add `--bridge 9877` to act on the open app.
4. **A cloud sandbox shell cannot reach the app.** A Linux sandbox can read and write project/script files (`.jsx`, `.ecproj`) but cannot run the user's `.exe`/app or reach its localhost port.
5. **Last resort: computer use on the UI** (slow). Keyboard shortcuts with `[` `]` may not work on non-US layouts (drag the layer bar end instead). Avoid double-clicking the title bar (it once returned to Home and closed the project). A new text layer starts small and as long as the comp.

## MCP tools

| Tool | What it does |
|---|---|
| `get_state` | Editor state (active comp, time, selection, tool, `dirty`) plus `app` info (version, command/effect counts, export formats). Cheap — use it to orient. |
| `get_project` / `get_comp {comp?}` / `doc_inspect` | Project items; comp settings and layers (id, index, name, type, in/out/start, blend, parent, matte). |
| `get_layer {layer, comp?, time?, depth?, flat?}` | Property tree; every node has a `path`. `flat: true` = compact list. |
| `get_property {layer, path, comp?, time?}` | Leaf value, keyframes, expression (`evaluated`, `expressionError`). |
| `set_property {layer, path, value?, time?, timeBase?, expression?, comp?}` | Static value; with `time` = keyframe; with `expression` ("" removes it). |
| `add_keyframe {layer, path, time+value \| keys:[...], timeBase?, interpolation?}` | Interpolation: linear, bezier, hold, easyEase, easyEaseIn, easyEaseOut. |
| `add_effect {layer, effect, values?}` | Apply and set values in one undo step; returns `effects/#n` and parameter paths. |
| `list_effects {filter?}` / `list_fonts {query?, variable?, rescan?}` | Effect registry; fonts (family + styles → `layer.setText {font, style}`). |
| `list_commands {filter?, enabled_only?, schemas?}` / `command_list` | Discover ids, parameters and whether they can run right now. |
| `describe_command {command}` | One command with its JSON Schema. Use when unsure: unknown keys are rejected (the error lists the accepted ones). |
| `execute_command {command, params?}` / `command_run {id, params?}` | Run a command. |
| `batch {steps:[{command, params}], label?, atomic?}` | Several commands = one undo; `"$N"` / `"$N.key"` refers to the result of step N (e.g. `"$1.layer"`); rolls back if a step fails. |
| `command_batch {steps, stop_on_error?}` | Several commands, one undo each, per-step results. |
| `run_script {code, name?}` | After Effects-style JavaScript; returns `{ok, result, output, error{message,line,column}}`. |
| `render_frame {comp?, time?, max_side?, path?, inline?, transparent?}` / `render_preview` | PNG of a frame to verify your work. |
| `open_project {path \| demo \| new}` / `save_project {path?}` | Open / save. |
| `undo {steps?}` / `redo` / `history {goto?}` | Branching history. |
| `script_ui {action, window?, widget?, value?, result?}` | Drive ScriptUI windows opened by scripts. |
| *(bridge only)* `screenshot {panel?, id?, max_side?, path?}`, `ui_inspect`, `ui_elements {prefix?}`, `ui_click`, `ui_drag`, `ui_key`, `ui_type`, `ui_set`, `control {method, params}` | See and operate the live window; `control` sends a raw control-channel method. |

Resources: `effectcraft://document`, `effectcraft://commands`.

**References.** Layers by id (from `get_comp`), `"#n"` (1-based from the top) or name; comps by id or name (default = active; `-` in the CLI). Ids may be strings (`"98"`). Property paths come from `get_layer`: `transform/position`, `transform/opacity`, `transform/scale`, `transform/rotation`, `effects/#1/blurriness`, `masks/#1/feather`, `contents/group/contents/gfill`, `@57` (uid). Times are in seconds; colors are `"#rrggbb"` or `[r,g,b,a]` in 0–1.

## Pitfalls (from the official docs)
- **Layer time vs comp time.** `add_keyframe`/`set_property` with `time` use *layer* time; `get_property`/`render_frame` use *comp* time. For moved or stretched layers pass `timeBase: "comp"`. In `run_script`, `setValueAtTime`/`keyTime` use comp time.
- **Moving or trimming a layer in time:** `layer.timing {layers:[...], start: 5.1}` (comp seconds, moves in and out together); also `in`, `out`, `delta`, `inDelta`, `outDelta`.
- **Name things when you create them.** `layer.newText|newShape|newSolid|newNull|newCamera|newLight` accept `name`; then use `"layer":"Title"`. A reference that matches nothing is an error.
- **Scripts.** `app.run(id, params)` calls any engine command from a script. A script that throws keeps the changes already made, so wrap it in `app.beginUndoGroup()` / `app.endUndoGroup()`. Scripts can only read inside the project folder; writing files/network needs Preferences ▸ Scripting & Expressions.
- **Locked layers are skipped.** Unlock with `layer.setSwitch {layers, switch:"lock", value:false}` or `layer.unlockAll`.
- **Unsaved changes.** Commands via MCP never prompt to save. In the app, `ui.menu.invoke` of commands that close a modified project shows Save / Don't Save / Cancel (`dialog.unsaved.save|dontSave|cancel`).
- **Missing footage after opening:** `footage.check {wait:true}` then `file.findMissing {what:"footage"}`.
- A command that changes nothing creates no undo step; default undo depth is 32 (`prefs.set {key:"general.undoLevels", value:99}`).
- To insert something "at the start", ask whether to overlap or push the rest later (`layer.timing` with `delta`).

## Recipes

Typical session:
1. `execute_command comp.new {"name":"Intro","width":1920,"height":1080,"frameRate":30,"duration":4}`
2. `execute_command layer.newText {"text":"Hello","size":160,"fill":"#ffffff","name":"Title"}` → `{"layer": 2}`
3. `add_keyframe {"layer":2,"path":"transform/position","keys":[{"time":0,"value":[-300,540]},{"time":1.5,"value":[960,540]}],"interpolation":"easyEase"}`
4. `add_effect {"layer":2,"effect":"Gaussian Blur","values":{"blurriness":8}}`
5. `render_frame {"time":1.0}` → `save_project {"path":"intro.ecproj"}`

| Goal | Calls |
|---|---|
| Orient | `get_state` → `get_project` → `get_comp {}` |
| Find the right command | `list_commands {"filter":"mask"}` → `describe_command {"command":"..."}` |
| Animate scale | `add_keyframe {"layer":2,"path":"transform/scale","keys":[{"time":0,"value":[0,0]},{"time":0.5,"value":[100,100]}],"interpolation":"easyEase"}` |
| Effect with settings | `list_effects {"filter":"glow"}` → `add_effect {"layer":2,"effect":"Glow","values":{"threshold":40,"radius":30}}` |
| Expression | `set_property {"layer":2,"path":"transform/rotation","expression":"time*90"}` |
| One undo step | `batch {"steps":[{"command":"layer.newShape","params":{"kind":"ellipse","name":"Ring"}},{"command":"layer.addShapeItem","params":{"layer":"$1.layer","kind":"trim"}},{"command":"prop.addKey","params":{"layer":"$1.layer","path":"transform/opacity","time":0,"value":0}}]}` |
| Keyframe velocity | `keys.select {"keys":[{"layer":"Ring","path":"contents/trim/end","time":0}]}` → `keys.velocity {"outSpeed":0,"outInfluence":33.33}` |
| Gradient | `set_property {"path":"contents/gfill/colors","value":["#0080ff","#ffff00"]}` |
| Mask | `layer.addMask {layer, shape: rect\|ellipse, rect}` → `layer.mask.set {field: feather\|expansion\|opacity, value}` |
| Layer order | `layer.arrange {"layers":[...], "above":"Name"}` (or `to: front\|forward\|backward\|back`, `index`) |
| Rename | `layer.rename {layer, name}` |
| Render video | `renderQueue.add {comp, format: h264\|prores\|webm\|png..., channels: rgba, proresProfile: 4444, output}` → `renderQueue.render {"wait": true}` |
| Lottie | `file.exportLottie {comp, path, includeExpressions}` → `{path, bytes, warnings}`; `file.importLottie {path}` |
| Premiere interchange | `file.importTimeline {path}` (xml/fcpxml/otio/edl/aaf/omf); `file.exportTimeline {comp, path, prerender?}` |
| Essential Graphics | `essential.set {layer, control:"Title", value}`; CLI `render --template lt.ectemplate --values v.json --out out.png` |
| Check your work | `render_frame {"time":1.5,"max_side":640}`; in bridge mode `screenshot {"panel":"Timeline"}` |
| Mistake | `undo {}` or `history {}` → `history {"goto": n}` |

**Text.** `layer.newText {text, size, fill, name, position | box, vertical?, edit?}` (`position` = point text, `box` = paragraph text). `layer.setText {layer, range?:[start,end], font, style, size, fill, tracking, kerning, baseline, allCaps, justify, indentLeft, spaceBefore...}` (no `range` = whole layer). Editing like the Type tool: `text.insert {text}`, `text.setSelection {start,end}`, `text.moveCaret {to, extend}`, `text.delete {word}`, `text.endEdit`. Fonts: `text.fonts` / `list_fonts`, `text.fontFeatures`; extra font folders via `EFFECTCRAFT_FONT_DIRS` (`;`-separated on Windows, `:` on macOS).

**Advanced (documented).** Tracking `track.motion|stabilize|new|setPoint|analyze {wait:true}|apply|options|status`; `track.mask {method, direction, wait}`, `mask.interpolate`; Warp Stabilizer `track.warpStabilizer`, `warp.analyze|status|cancel`; 3D camera tracker `track.camera`, `camera.solveStatus|points|selectPoints|setGroundPlane|createFromSolve|deletePoints|create`; Roto Brush `roto.stroke|propagate|span|freeze|unfreeze|clearStrokes|status|models|model.download|model.select`; Object Matte `objectMatte.apply|propagate|freeze|status`; Puppet `puppet.addPin|recordPin|selectPins|setPin|follow`, `paths.pointsFollowNulls|nullsFollowPoints`; WebAssembly effect plug-ins `effect.plugins.load {path}` / `effect.plugins.list`; ScriptUI `scriptui.list|get|click|set|close|mouse`, `file.scripts.list`, `file.runScript {name}`; environment light `layer.newLight {kind:"Environment"}` + `lightOptions/source`; `prop.pickWhip {layer, prop, target}`, `prop.reset`, `keys.toggleTransform {prop}`, `layer.quickOffset {delta}`, `keys.quickOffset`, `keys.copy|paste`, `effect.copy|paste`, `effect.pickColor`, `project.select`, `project.usedIn {item}`, `jobs.list`.

## CLI reference

Every call starts a headless engine: it opens the demo project unless you pass `--project F.ecproj`, a positional `*.ecproj`, or `--empty`. `--json` = one JSON document on stdout; errors are `{"error":...}` with exit 1; usage errors exit 2.
```
effectcraft-cli <info|commands|exec|run|props|get|set|render-frame|render|script|mcp|bench> [args] [--json]
options: --project F.ecproj | --demo | --empty   --save | --save-as F   --bridge PORT   --json   --gpu
```
```sh
effectcraft-cli info --json
effectcraft-cli commands --filter keys
effectcraft-cli exec --list --schemas --json
effectcraft-cli exec comp.new --params '{"name":"Main","width":1280,"height":720}' --empty --save-as main.ecproj
effectcraft-cli exec layer.newSolid '{"color":"#3366ff"}' main.ecproj --save
effectcraft-cli props Main '#1' --project main.ecproj
effectcraft-cli set Main '#1' transform/position '[100,360]' --time 0 main.ecproj --save
effectcraft-cli get Main '#1' transform/position --time 0.5 main.ecproj --json
effectcraft-cli run main.ecproj comp.open '{"comp":"Main"}' time.set '{"time":1}' --json
effectcraft-cli render-frame main.ecproj --time 1 --max-side 640 --out f.png --json
effectcraft-cli render --comp Main --out main.mp4
effectcraft-cli exec layer.newNull --bridge 9877            # same commands against the live app
effectcraft-cli script build.jsx --save-as main.ecproj
effectcraft-cli script --eval 'app.project.item(1).numLayers' main.ecproj --json
effectcraft-cli script tweak.jsx --bridge 9877
```
`render` is headless; in bridge mode use `exec renderQueue.add` + `renderQueue.render`. In Windows PowerShell single quotes work; call the exe with `&` when its path is quoted. On macOS/Linux shells use the same quoting as above.

## Control channel (raw protocol)

TCP on `127.0.0.1:<port>` only, one JSON object per line each way, persistent connections, max 16, 60 s timeout, **no authentication** (enable it only while you need it).
- Request: `{"id":1,"method":"engine.execute","params":{"command":"comp.new","params":{"name":"A"}}}`
- Response: `{"id":1,"ok":true,"result":...}` or `{"id":1,"ok":false,"error":"..."}`
- An invalid line closes the connection.

| Method | Use |
|---|---|
| `engine.execute {command, params?}` | Any command, as headless (never opens dialogs: `comp.new {}` makes a default comp). UI ids (`tool.*`, `view.*`, `window.*`, `playback.*`, `timeline.*`, `app.*`) go to the menu. |
| `ui.menu.invoke {id, params?}` | Like a menu click (may open dialogs). |
| `engine.commands {filter?, enabledOnly?}`, `ui.menu.list` | Listings. |
| `render.frame {comp?, time?, max_side?, path?, base64?}` | PNG of a frame. |
| Queries (not undoable): `project.summary`, `comp.info`, `layer.tree {layer, depth?, time?}`, `prop.get`, `editor.state`, `command.list`, `command.describe`, `app.capabilities`, `jobs.list` | |
| `ui.inspect`, `ui.elements {prefix?}`, `ui.set {tool?, workspace?, viewer?, timeline?}`, `ui.panel.show\|close {panel}`, `ui.playback {action}`, `ui.resize`, `ui.focus`, `app.quit {force?}` | UI state. |
| `ui.click {id \| x,y, button?, count?, modifiers?}`, `ui.move`, `ui.drag {from, to}`, `ui.scroll`, `ui.key {key}`, `ui.type {text}` | Input (target `{id}` or `{x,y}` in points). |
| `ui.viewer.locate\|hit`, `ui.timeline.locate {layer, time}` | Coordinate conversions. |
| `ui.screenshot {path?, panel?, id?}` | Window / panel / element screenshot. |

Stable element ids: `tools.Selection`, `panel.Timeline`, `panel.tab.EffectControls`, `project.item.<id>`, `viewer.comp`, `timeline.layer.<id>.bar|row|twirl`, `timeline.prop.<uid>.stopwatch`, `timeline.key.<uid>.<frame>`, `timeline.cti`, `effectControls.prop.<uid>.value`, `effects.item.<name>`, `dialog.unsaved.*`. Recommended loop: `ui_elements` → `ui_click`/`ui_drag` → `screenshot {panel:"Timeline"}`.

## After Effects-style scripting
```js
app.beginUndoGroup("Intro");
var c = app.project.items.addComp("Main", 1920, 1080, 1, 5, 30); // name, w, h, pixelAspect, duration, fps
c.motionBlur = true; c.bgColor = [0.08, 0.04, 0.03];
var t = c.layers.addText("hello");
var st = t.property("Text").property("Source Text");
var d = st.value; d.font = "Inter SemiBold"; d.fontSize = 200; d.fillColor = [1, 0.9, 0.7];
d.justification = ParagraphJustification.CENTER_JUSTIFY; d.tracking = 50; st.setValue(d);
t.inPoint = 0; t.outPoint = 2;
var pos = t.property("Transform").property("Position");
pos.setValueAtTime(0, [960, 700]); pos.setValueAtTime(1, [960, 540]);
var e = new KeyframeEase(0, 85); pos.setTemporalEaseAtKey(2, [e], [e]);
t.property("Transform").property("Opacity").expression = "linear(time,0,1,0,100)";
t.property("Effects").addProperty("ADBE Gaussian Blur 2");
app.run("layer.new3dPrimitive", {kind: "cube"});   // any engine command
app.endUndoGroup();
writeLn("ok");
```
- More: `layers.addSolid(color, name, w, h, pa)`, `addNull()`, `addShape()`, `layers.add(comp)` (precomp, `L.startTime`), `trackMatteType = TrackMatteType.ALPHA`, `blendingMode = BlendingMode.ADD`, `parent`, `sourceRectAtTime`, `app.project.save(new File("x.ecproj"))`.
- Shapes: `Contents.addProperty("ADBE Vector Group")` → `ADBE Vector Shape - Rect|Ellipse|Group`, `ADBE Vector Graphic - Fill|Stroke|G-Fill`, `ADBE Vector Filter - Trim|Repeater|Roughen|Wiggler`.
- Text animators: `ADBE Text Animator` + `ADBE Text Opacity|Position 3D|Scale 3D|Blur|Tracking Amount|Rotation|Fill Color`, selector `ADBE Text Selector`.
- Match names that work: `ADBE Gaussian Blur 2`, `ADBE Ramp`, `ADBE Glo2`, `ADBE Fractal Noise`, `ADBE Drop Shadow`, `ADBE Tint`, `ADBE Linear Wipe`, `ADBE Fill`, `CC Vignette`.
- Tip: when converting CSS `cubic-bezier` easings, write a small helper that maps the control points to AE ease (speed/influence) pairs.

## Engine commands by menu (v0.9.x)
From the menu map; for exact parameters use `describe_command`. If an id no longer exists: `list_commands {filter}`.

**File:** `file.newProject`, `file.newFromTemplate`, `project.newFolder`, `file.open`, `file.openDemoProject`, `file.close`, `file.closeProject`, `file.save`, `file.saveAs`, `file.saveCopy`, `file.saveCopyAsXml`, `templates.saveAs`, `file.incrementAndSave`, `file.revert`, `file.import {paths, sequence?}`, `file.importMultiple`, `file.importTimeline`, `file.importPlaceholder`, `file.importSolid`, `file.importLottie`, `file.importVanishingPoint`, `essential.importTemplate`, `renderQueue.add`, `file.exportTimeline`, `file.exportLottie`, `essential.exportTemplate`, `app.find`, `layer.addItem`, `file.newCompFromSelection`, `file.collectFiles`, `file.consolidateFootage`, `file.removeUnusedFootage`, `file.reduceProject`, `file.findMissing {what: effects|fonts|footage}`, `file.relinkFootage`, `file.watchFolder`, `file.installScript`, `file.installScriptUIPanel`, `file.runScript`, `file.createProxy {kind: movie|still}`, `file.setProxy`, `file.setProxyNone`, `file.interpretFootage {items, frameRate?, startTimecode?, preserveRgb?}`, `file.interpretProxy`, `file.rememberInterpretation`, `file.applyInterpretation`, `file.replaceFootage`, `file.replaceWithPlaceholder`, `file.replaceWithSolid`, `file.replaceWithLayeredComp`, `file.reloadFootage`, `file.revealInFinder`, `file.projectSettings {timeDisplay?}`, `app.settings {page}`, `app.quit`.

**Edit:** `edit.undo`, `edit.redo`, `app.commandPalette`, `edit.cut`, `edit.copy`, `edit.copyWithPropertyLinks`, `edit.copyWithRelativePropertyLinks`, `edit.copyExpressionOnly`, `edit.paste`, `edit.pasteReversedKeyframes`, `edit.pasteTextMatchFormatting`, `edit.pasteTextFormattingOnly`, `edit.clear`, `edit.duplicate`, `edit.splitLayer`, `edit.liftWorkArea`, `edit.extractWorkArea`, `edit.selectAll`, `edit.deselectAll`, `edit.selectLabelGroup`, `edit.label {label}`, `edit.purge {what}`, `edit.purgeUndo`, `edit.editOriginal`, `edit.history.list`, `edit.history.goto`, `app.keyboardShortcuts`.

**Composition:** `comp.new`, `comp.settings {preserveFrameRate?, preserveResolution?...}`, `comp.info`, `comp.open`, `comp.setPosterTime`, `comp.trimToWorkArea`, `comp.cropToRegionOfInterest`, `comp.cropToLayerBounds`, `render.addOutputModule`, `playback.toggle`, `playback.cacheWhenIdle`, `playback.audio`, `comp.saveFrameAs`, `comp.saveFrameAsPsd`, `comp.saveFrameAsExr`, `comp.copyFrame`, `render.preRender`, `render.saveCurrentPreview`, `comp.openInEssentialGraphics`, `comp.responsiveTime {op: intro|outro|workArea}`, `comp.flowchart`, `comp.miniFlowchart`, `comp.vr.createEnvironment`, `comp.vr.extractCubemap`, `comp.revealInProject`.

**Layer — new:** `layer.newText`, `layer.newSolid {color, name?}`, `layer.newLight {kind?}`, `layer.newCamera`, `layer.newNull`, `layer.newShape {kind?, name?}`, `layer.newAdjustment`, `layer.newContentAwareFill`, `layer.new3dPrimitive {kind: cube|sphere|plane|torus|cone|cylinder, position?}`.
**Layer — general:** `layer.select {layers, add?, toggle?}`, `layer.rename {layer, name}`, `layer.timing {layers, start|in|out|delta|inDelta|outDelta}`, `layer.settings {layer, name?, color?, width?, height?}`, `layer.setText`, `layer.setParent`, `layer.arrange`, `layer.openLayer`, `layer.openSource`, `layer.revealSource`, `layer.revealExpressionErrors`, `layer.environment`, `layer.autoTrace`, `layer.precompose`, `layer.sceneEditDetection`, `layer.addShapeItem {layer, kind, group?}`, `layer.channels`.
**Masks / paths:** `layer.addMask`, `layer.mask.shape`, `layer.mask.set {field, value}`, `layer.mask.reset`, `layer.mask.remove`, `layer.mask.removeAll`, `layer.mask.mode {mode: None|Add|Subtract|Intersect|Lighten|Darken|Difference}`, `layer.mask.invert`, `layer.mask.lock`, `layer.mask.motionBlur {mode}`, `layer.mask.featherFalloff {mode}`, `layer.mask.unlockAll`, `layer.mask.lockOthers`, `layer.mask.hideLocked`, `mask.insertVertex|convertVertex|deleteVertices`, `mask.setClosed`, `path.rotoBezier`, `path.convertToBezier`, `path.setFirstVertex`, `path.freeTransform`, `path.groupShapes`, `path.ungroupShapes`, `shape.newPath`, `shape.newShape`.
**Quality / switches:** `layer.quality {quality: best|draft|wireframe}`, `layer.sampling {sampling}`, `layer.setSwitch {layers?, switch: shy|lock|video|audio|solo|collapse|fx|motionBlur|adjustment|threeD|guide|preserveTransparency, value?}`, `layer.hideOtherVideo`, `layer.showAllVideo`, `layer.unlockAll`, `layer.expressions {enabled}`.
**Transform:** `layer.transform {op: reset|center|fit|fitWidth|fitHeight|flipH|flipV}`, `layer.setTransform {prop}`, `layer.centerAnchor`, `layer.autoOrient`.
**Time:** `layer.enableTimeRemap`, `layer.timeReverse`, `layer.timeStretch`, `layer.freezeFrame`, `layer.freezeOnLastFrame`, `layer.alignVideoToData`, `layer.frameBlending {mode: off|frameMix|pixelMotion}`, `layer.quickOffset`.
**Markers:** `layer.addMarker`, `comp.addMarker`, `layer.updateMarkersFromSource`, `layer.markersLock`, `layer.deleteAllMarkers`.
**Blending / mattes:** `layer.setBlendMode {mode: Normal|Dissolve|Darken|Multiply|Color Burn|Linear Burn|Add|Lighten|Screen|Color Dodge|Linear Dodge|Overlay|Soft Light|Hard Light|Linear Light|Vivid Light|Pin Light|Hard Mix|Difference|Exclusion|Subtract|Divide|Hue|Saturation|Color|Luminosity|Stencil Alpha|Stencil Luma|Silhouette Alpha|Silhouette Luma|Alpha Add|Luminescent Premul|...}` or `{step: 1|-1}`, `layer.trackMatte {op: none|alpha|alphaInverted|luma|lumaInverted|above|below}`.
**Styles:** `layer.style.options|convertToEditable|showAll|removeAll|dropShadow|innerShadow|outerGlow|innerGlow|bevelEmboss|satin|colorOverlay|gradientOverlay|stroke`.
**Create:** `layer.create {op: shapesFromText|shapesFromVector|masksFromText|editableText|instance3d|keyframesFromData|nullTracing|nullsFollowing|nullsPathPoints|nullsPositional}`.
**Camera / lights / materials:** `camera.fromView`, `camera.stereoRig`, `camera.orbitNull`, `camera.fromModel`, `camera.linkFocusToPoi`, `camera.linkFocusToLayer`, `camera.setFocusToLayer`, `layer.cameraSettings`, `light.fromModel`, `light.controlWithCamera`, `light.environmentBackground`, `material.revealSource`, `material.reset`, `material.duplicateAssign`.

**Effects:** `effect.apply {layer, effect}` (→ `{effects, paths}`), `effect.applyLast`, `effect.removeAll`, `effect.manage`, `effect.list`, `effect.copy`, `effect.paste`, `effect.pickColor`, `effect.warning`, `effect.setLabel`, `effect.plugins.load|list`. Ids look like `ec.blur.gaussian`, `ec.color.levels`, `ec.stylize.glow`, or use After Effects names (`Gaussian Blur`, `Glow`, `Curves`, `Keylight (1.2)`).

**Animation:** `anim.savePreset`, `anim.applyPreset`, `anim.browsePresets`, `layer.applyTextPreset {preset: typewriter|fadeUpCharacters|blurIn|trackingIn|bounceInWords|dropInLines|scramble|jitter}`, `anim.addKeyframe`, `prop.get`, `prop.set`, `prop.addKey`, `prop.setExpression`, `prop.reset`, `prop.pickWhip`, `prop.separateDimensions`, `prop.convertExpressionToKeyframes`, `keys.select {keys, toggle?}`, `keys.selectAll`, `keys.selectEqual`, `keys.set`, `keys.move {delta, merge?}`, `keys.transform`, `keys.copy`, `keys.paste`, `keys.delete`, `keys.toggleHold`, `keys.toggleTransform {prop}`, `keys.interpolation`, `keys.velocity {inSpeed?, inInfluence?, outSpeed?, outInfluence?}`, `keys.easyEase`, `keys.easyEaseIn`, `keys.easyEaseOut`, `keys.exponentialScale`, `keys.audioToKeyframes`, `keys.timeReverse`, `keys.quickOffset`, `keys.setSpatialTangents`, `layer.sequence`, `layer.enablePerChar3D`, `layer.addTextAnimator {properties:[...]}`, `text.addSelector {kind: range|wiggly|expression}`, `text.removeAllAnimators`, `text.animatorFontAxes`, `essential.addProperty`, `essential.set`, `anim.reveal {kind}`.

**View / time:** `time.set {time}`, `time.nextKey`, `time.previousKey`, `time.workAreaStart|End`, `view.res.full|half|third|quarter|auto {comp?}`, `view.res.custom {factor}`, `view.zoomIn|zoomOut`, `view.newViewer`, `view.rulers`, `view.guides`, `view.addGuide`, `view.clearGuides`, `view.grid`, `view.snapping`, `view.snappingOptions`, `view.channel`, `view.exposure`, `view.takeSnapshot`, `view.showSnapshot`, `view.fastPreviewMode`, `view.setRegionOfInterest`, `view.layout {views: 1|2|4}`, `view.3d.activeCamera|default|front|left|top|back|right|bottom|custom1-3|last`, `view.reset3DView`, `view.lookAtSelected|lookAtAll`, `view.fullScreen`.

**Window / help:** `window.workspace {name: Default|Standard|Small Screen|Animation|Effects|Text|Review|Learn|Minimal|Color|Paint|Motion Tracking|Essential Graphics|All Panels}`, `window.panel {panel}`, `window.saveWorkspace|saveWorkspaceAs|resetWorkspace|editWorkspaces`, `window.floatPanel`, `window.dockAction`, `timeline.column`, `timeline.search {query}`, `timeline.reveal.<kind>`, `timeline.twirlSelected`, `timeline.collapseAll`, `flowchart.graph`, `prefs.set {key, value}`, `help.docs {page: help|effects|expressions|scripting}`, `help.systemReport`, `help.enableLogging`, `help.revealLogFile`, `app.about`.

## Recommended workflow
1. Quick-check the connection; `get_state` to see whether a project is open and `dirty`.
2. Work on a copy of the user's project (or a new comp) unless they say otherwise.
3. Build with `batch` for multi-step changes; name every layer; verify with `render_frame` after each visible change.
4. Save (`save_project`) or render only with the user's OK, and never overwrite their original silently.
5. Close the control channel (quit the "(Claude)" launcher instance) when finished — it has no authentication.
