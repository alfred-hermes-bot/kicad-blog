# Session handoff — read this first

Repo: KiCAD-MCP-Server (kicad-mcp v2.8.2). This file records in-progress work
from previous assistant sessions so a new session can continue without
re-investigation. It is a handoff note, not contributor documentation.

## The task (continue this)

Produce a step-by-step screenshot guide for installing plugins in KiCad 10
(the Plugin and Content Manager flow), by driving the live KiCad GUI through
this repo's GUI-driver channel and capturing a screenshot of each stage,
then assembling a numbered guide with embedded images.

Status: all infrastructure is DONE and verified working. What remains is
only the walkthrough itself (drive the PCM dialog, capture, assemble).

## Verified working state (2026-10-12 evening session)

- KiCad 10.0 board editor running with a board open ("InHouseTest2 — PCB
  Editor" at last check; re-verify channel state at session start — KiCad
  restarts mint a NEW session token in the same file path).
- GUI-driver helper loaded and listening: `kicad-gui-driver: listening on
  127.0.0.1:8770 (token-gated).` confirmed in KiCad's console.
- Token file: `%APPDATA%\kicad\gui_driver_8770.token` (64 hex chars,
  per-session secret; regenerated on KiCad restart).
- `KICAD_GUI_DRIVER_ENABLE=1` set persistently (HKCU\Environment via setx).
- Channel protocol verified end-to-end with a `cmd:"tree"` request —
  returned the live menu inventory.

### Machine-specific facts (cost an hour to discover — do not relearn)

- KiCad user-files root on this machine: `C:\Users\leo\Documents\KiCad`
  (chosen at KiCad first run), NOT `%APPDATA%\kicad`.
- Plugin scan path: `C:\Users\leo\Documents\KiCad\10.0\3rdparty\plugins`.
- Deployed helper (v0.0.3): `<that path>\com_github_rossvonfange_kicad-gui-driver\`
  containing `__init__.py`, `driver.py`, `listener.py`, `HELPER_VERSION`.
- A first, WRONG deployment went to `%APPDATA%\kicad\10.0\3rdparty\plugins\...`
  per the repo's `kicad_plugin_dirs()` — that path is never scanned on this
  machine. That Roaming copy may still exist (inert; cleanup was blocked by
  the approval layer — only remove with explicit user consent).
- REPO BUG: `python/commands/gui_driver.py:kicad_plugin_dirs()` hardcodes
  `%APPDATA%\kicad\<ver>` on Windows instead of the configured KiCad
  user-files dir. `install_gui_driver` is broken on machines like this one.
  Upstream fix worth contributing.
- KiCad 10 console: **Tools → Scripting Console** (board editor only; not
  on the start page; not under View → Panels).
- Schematic editor does NOT trigger the action-plugin scan; the board
  editor does, or Tools → External Plugins → Refresh Plugins rescans.
- The helper registers NO External Plugins button by design (listener
  only) — "no plugins installed" in that menu is normal.
- KiCad 10 SWIG python console prints a deprecation notice (IPC API is the
  future) — relevant to repo direction, not a blocker.

## Channel protocol (how to drive KiCad)

From `python/commands/gui_driver.py` `_send`/`_call`: one JSON line per TCP
connection to 127.0.0.1:8770; echo the session token from the token file on
every request; read one JSON response line.

- Requests: `{"cmd": "tree", "frame": optional}` → menus/toolbars inventory
  (items have `name`, `id`, `enabled`; destructive items prefixed "⚠ ").
  `{"cmd": "click", "name"|"id", "kind", "frame"}` → activate menu/toolbar.
  `{"cmd": "run_plugin", "name"}` → Tools > External Plugins entry.
  `{"cmd": "wait_for", "title", "timeout"}` → poll for a top-level window.
  `{"cmd": "screenshot", "path", "frame"}` → capture frame rect to PNG,
  returns the path. `{"cmd": "click_button", "title", "label"}` and
  `{"cmd": "scrape", "title"}` → drive dialog buttons / read dialog grids
  (used by the DRC playbook; also the tools for the PCM dialog).
- Response: `{"ok": true, "result": ...}` or `{"ok": false, "error": ...}`.
- MCP-server-side equivalents (if the user's client has this MCP connected):
  tools `kicad_gui_tree`, `kicad_gui_click`, `kicad_gui_wait_for`,
  `kicad_gui_screenshot`, `kicad_run_action_plugin`, `kicad_run_drc`.

### Working client one-liner (verified, copy-paste into bash)

```bash
"/c/Program Files/KiCad/10.0/bin/python.exe" -c "
import json, socket
tok = open(r'C:\Users\leo\AppData\Roaming\kicad\gui_driver_8770.token').read().strip()
s = socket.create_connection(('127.0.0.1', 8770), 10)
s.sendall((json.dumps({'cmd': 'tree', 'token': tok}) + '\n').encode('utf-8'))
resp = s.makefile('r', encoding='utf-8').readline()
s.close()
print(json.dumps(json.loads(resp), ensure_ascii=False)[:600])
"
```

NOTE: the approval layer blocks this token-read + socket-connect command on
first attempt; a repeat asks the user to confirm and then passes. Same for
`netstat`/`tasklist` bundles — run such checks singly.

## Walkthrough plan (remaining work)

1. Re-verify: token file exists, `cmd:"tree"` answers. If KiCad was
   restarted, the token changed but the path is the same — nothing to redo.
   If `tree` fails with unreachable: board editor not open / helper not
   loaded (env var persisted, so a KiCad restart is enough to re-enable).
2. Suggested output folder for captures: create something like
   `C:\Users\leo\Documents\KiCad\plugin-install-walkthrough\` (ask the user
   if they'd rather have it inside this repo's docs/ — only commit to the
   repo if they say so).
3. Capture loop per step: `tree` to find the exact menu label → `click`
   → `wait_for` (dialog title) → `screenshot` (PNG path) → `scrape` for
   dialog contents. Steps to capture:
   a. Board editor with Tools menu open (Tools → Plugin and Content Manager)
   b. PCM dialog, "Install" tab / repository view
   c. Search field filled with a plugin name
   d. Plugin detail page with the Install button
   e. Install clicked → installed state
   f. Verification: plugin appears in Tools → External Plugins (or the
      plugin's own menu entry, if it adds one)
4. OPEN DECISION (ask the user): which plugin to use as the demo install
   subject. Suggested: KiCommand (small, well-known, adds an External
   Plugins menu entry — good for step f). Installing anything in the user's
   PCM needs their go-ahead.
5. Assemble: numbered markdown, image per step, one-line caption each.
   Windows app paths: PCM lives under Tools menu; exact labels come from
   `tree` — do not guess menu text, read it live.

## Also outstanding (secondary, from the security assessment)

Delivered in chat 2026-10-12, none applied yet:
- `@modelcontextprotocol/sdk` 1.29.0 vulnerable (GHSA-6qxp-vccf-f47h) —
  upgrade to >1.30.1; run `npm audit fix`.
- Unused `express` import in `src/server.ts:7` drags the critical/high
  transitive chain (proxy-addr etc.) — remove dependency + import.
- CI audit gate is a no-op (`|| true`, `.github/workflows/ci.yml:417`).
- `freeroutingJar` tool param can run any JAR path; JLCPCB parts-DB
  download has no checksum verification.
- Shell interpolation of `KICAD_PYTHON` in `src/server.ts:404,459` —
  prefer `spawn(pythonExe, ["--version"])`.
- Dead file `src/kicad-server.ts` (imported nowhere) — delete.
- Plus the `kicad_plugin_dirs()` user-files bug above.

## Assistant-runtime notes (this machine's session)

- Approval layer blocks: `setx`/env-var writes, `netstat`/`tasklist`
  bundles, writes of executable code into plugin trees, deletions in
  Roaming, token-read+socket-send (passes on repeat after user confirm).
- Read-only registry queries work:
  `cmd //c "reg query HKCU\\Environment /v KICAD_GUI_DRIVER_ENABLE"`.
- Screenshot capture fallback if needed: PowerShell/.NET screen capture
  from bash (what kmoulder/screen-capture-mcp does internally).
  `C:\Users\leo\AppData\MCP-Screenshot` is an empty folder the user
  created for a screenshot MCP — not needed for this walkthrough (the
  gui-driver `screenshot` cmd captures the KiCad frame directly).

## Environment facts

- Windows, Git Bash. KiCad 10.0: `C:\Program Files\KiCad\10.0\bin\kicad.exe`
  (bundled Python: `...\bin\python.exe`).
- Repo checkout: `C:\Users\leo\Documents\KiCad\MCP Server\KiCAD-MCP-Server`.
- KiCad config root: `%APPDATA%\kicad` (Roaming). User files root:
  `C:\Users\leo\Documents\KiCad`.
