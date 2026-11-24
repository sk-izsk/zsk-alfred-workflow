# Project layout (Bun + TypeScript, everything generated from code)

Reference projects: `z-bitwarden` and `z-vscode` (github.com/sk-izsk/z-vscode-workflow). Copy their
`scripts/`, `ui/shared.ts`, `src/ui.ts`, `src/env.ts`, `src/proc.ts` and the launcher when starting.

```
src/              runtime (bundled to workflow/bin/<cli>.mjs, runs on Node inside Alfred)
  main.ts         router: `<cli> <command> args…` → command; UI commands print one JSON action
  ui.ts           JSON action builders: update, fn, vars, finish, popup, sheet, status, script…
  env.ts          config (env → info.plist defaults + prefs.plist), data/cache dirs, atomic writes
  proc.ts         run/exec (no shell), osascript with argv (never splice strings into AppleScript)
  commands/       one file per window or menu; returns actions
ui/               windows as TypeScript objects → workflow/<name>.alfredui/ui.json
  shared.ts       building blocks: searchBar, list, row, pill, footer, hiddenKey, picker, window…
  index.ts        { "<name>.alfredui": windowObject }
scripts/
  plist.ts        info.plist as code: objects, connections, uidata, userconfigurationconfig, readme
  build-workflow.ts  writes every ui.json + info.plist (plutil -convert xml1) + icons
  icons.swift     original icons rendered from SF Symbols (never ship someone else's logo)
  link.sh         symlink workflow/ into Alfred's preferences + reload (dev loop)
  package.sh      zip workflow/ → dist/<Name>.alfredworkflow, excluding prefs.plist, .DS_Store, tool dirs
workflow/         what Alfred loads: info.plist, icon.png, <uid>.png object icons, bin/, *.alfredui/
tests/            bun:test for parsers, ordering, routing, file edits
```

`package.json` scripts: `build` (bun build src/main.ts --target=node → workflow/bin/<cli>.mjs, then
build-workflow), `check` (tsc --noEmit, oxlint, oxfmt --check, bun test), `link`, `package`.
Conventions: Bun for packages, arrow functions only, small modules, oxlint + oxfmt, no runtime deps
(Node stdlib + macOS tools). `.gitignore`: node_modules, dist, the bundle, `workflow/prefs.plist`.

## The launcher (`workflow/bin/<cli>`)

A `#!/bin/zsh --no-rcs` script that finds Node (`$node_path` setting, else read it from prefs.plist,
then /opt/homebrew, /usr/local, ~/.volta, ~/.local, newest ~/.nvm and ~/.local/share/fnm, then bun) and
`exec`s the bundle; if none is found it prints a JSON status update explaining how to fix it.

## info.plist objects (types and the config keys that matter)

- `input.keyword` v1: keyword `{var:kw_x}`, text, subtext, argumenttype 0 required / 1 optional / 2 none.
- `input.scriptfilter` v3: keyword, script `./bin/cli root "$1"`, type 11 (zsh), escaping 102,
  argumenttype 1, alfredfiltersresults false, queuemode 1, queuedelayimmediatelyinitially true.
- `input.filefilter` v2: keyword, fields (kMDItemDisplayName…), types, scopes, anchorfields.
- `trigger.hotkey` v2 (leave hotkey unset: hotkey 0, hotmod 0, hotstring ""), `trigger.external` v1.
- `trigger.action` v1 (File Action: filetypes, acceptsmulti, name).
- `trigger.universalaction` v1: acceptsfiles/acceptstext/acceptsurls, acceptsmulti, name.
- `utility.json` v1 launch mode before a window:
  `{"alfredworkflow":{"arg":"{query}","config":{"launchmode":"{var:launch_external}"}}}`.
- `userinterface.alfredui` v1: path `name.alfredui`, stackview true, launchmode 0.
- `utility.conditional` v1 to route Script Filter items by a variable (`{var:target}`), one output per
  target, `elselabel` for actions.
- `action.script` v2 (type 11, scriptargtype 1), `output.notification` v1 (text `{query}`,
  onlyshowifquerypopulated).
- Connections: `{destinationuid, modifiers, modifiersubtext, sourceoutputuid?, vitoclose}`.
  Modifiers: ⌘ 1048576, ⌥ 524288, ⌃ 262144, ⇧ 131072. A Script Filter item's `mods.cmd` needs its own
  ⌘ connection.
- Object icons: copy `<uid>.png` into workflow/.
- Configuration Builder entries (`userconfigurationconfig`): textfield, textarea, checkbox,
  popupbutton (picker), filepicker (filtermode 1 = folders), slider. Values are always strings
  (checkbox "1"/"0").

## Shape of a good workflow

- **One main keyword** opens a Script Filter menu of all commands; typing filters it; extra words go
  to the command (`kw install foo`). Short keywords for frequent commands, all configurable.
- Script Filter item `variables` route to windows/actions; `autocomplete` drills into sub-lists.
- Windows for rich lists; a Run Script after each window handles "back" and hand-offs.
- Universal Actions for anything that works on files/text/URLs anywhere in Alfred.

## Testing

- Unit: `bun test` for pure logic. Integration: run each command from its window folder in a bare env
  and inspect the JSON (row counts, status text, keys). Back up any user file a test touches and
  `cmp` it afterwards.
- On screen: reload, then ask the user (or `screencapture -l`) to confirm. Demo data for screenshots:
  temp folders + a hidden setting that points the data source elsewhere; restore prefs after.

## Releasing

Bump `version` (goes into info.plist) → `bun run check` → `bun run package` → `unzip -l` to verify no
prefs.plist/tool folders/personal paths → merge → `gh release create vX.Y.Z <file>.alfredworkflow`.
