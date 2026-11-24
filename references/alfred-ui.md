# Alfred UI cheat sheet (Alfred 6)

Condensed from the docs (see `docs.md`) plus patterns proven in working workflows. When a detail
matters, confirm it on the doc page: this toolkit is new and the docs have small typos.

## Shape

- A window is a folder ending in `.alfredui` containing `ui.json`; paths inside are relative to it
  (scripts usually `../bin/<cli>`). The workflow's **Alfred UI** object points at that folder.
  `stackview: true` makes ⎋ return to the previous view.
- `ui.json` top level: `bundleid` (needed for persisted variables), `variablesconfig`, `variables`,
  `window`, `log`, `pulse`, `colours`, `templates`.
- `window`: `title`, `size` {width,height}, `minsize`, `root` (the root view), `appearance`
  (automatic|light|dark), `blur`, `toolbar`… Inside Alfred, width defaults to the theme width; a fixed
  width in ui.json can stop the window opening inside Alfred, so resize after load:
  `on("preload", json({ window: { resize: { width: 760 } } }))`.

## Components

`view` (layout: `inline` with `direction`, `border` with child `constraints.region`
left|right|top|bottom|centre, `form`, `stack`), `label`, `button`, `checkbox`, `picker`, `textfield`,
`passwordfield`, `textarea`, `markdownarea`, `imageview`, `progress`, `dataview`, `webkitview`.

Shared properties: `id` (globally unique), `constraints` {region, position, size, insets, stretch},
`events`, `focus` (one per UI), `hidden`, `opacity`, `enabled`, `tooltip`, `menu`, `shortcut`
(AppKit key equivalent on buttons/checkboxes/pickers, e.g. `"⌘S"`, `"⇧⌘↩"`, `"↩"`, `"⎋"`).

- **label**: `text`, `colour` (`labelColor`, `secondaryLabelColor`, `systemGreenColor`…), `font`
  {size, weight, monospaced}, `textalign`.
- **imageview**: `image` {type: `file` | `fileicon` (Finder icon of a path) | `filetypeicon` |
  `symbol` (SF Symbol), src, size}, `contenttintcolour`, `resize`. **No URL images**: download first.
- **picker**: `items` [{text, value, type: item|separator}], `selected` (a value), `changed` event,
  `{value:picker.selected.value}`.
- **textfield**: `text`, `placeholder`, `bordered`, `focusring`, `font`; events `changed`,
  `typingstopped`, `actioned` (↩).
- **webkitview**: `src` (URL or file) or `text` (HTML string), `openlinksexternally`; functions back,
  forward, reload.
- **dataview** (the list):
  ```json
  { "id": "items", "type": "dataview", "placeholder": "No results",
    "data": { "type": "basic", "src": [],
              "filter": { "type": "basic", "args": "match", "match": "{value:searchField.text}",
                          "matchmode": "wordmatchanyorder" } },
    "cell": { "height": 38, "root": { …cell view… },
              "init": { "rowCell.background": null, "rowTitle.colour": "labelColor" },
              "selected": { "rowCell.background": "alfredSelectedResultBackgroundColor",
                            "rowTitle.colour": "alfredSelectedResultTextColor" } } }
  ```
  Rows are objects keyed `componentId.property` inside the cell, plus `uid` and any filter field:
  `{ "uid": "id1", "rowTitle.text": "Name", "rowIcon.image.type": "fileicon", "rowIcon.image.src": "/path",
  "rowPill.hidden": true, "match": "searchable words" }`.
  Functions: `filterdata`, `selectfirst`, `selectnext`, `selectprevious`, `scrolltop`…
  Selected row: `{value:items.selected.uid}`; inside a cell event: `{data:uid}`.
  Theme colours `alfredSelectedResultBackgroundColor` / `alfredSelectedResultTextColor` make the
  selection match the user's Alfred theme.

## Events and actions

- `events: [{ "triggers": "changed" | "actioned" | "typingstopped" | "launched" | "preload" | "quit" |
  "clicked" | "doubleclicked", "action": ACTION | [ACTION…], "confirm"?: {…} }]`.
- Action types: `script` {src, args, async, variables}, `json` {src: JSON-actions, variables},
  `function` {src: componentId, args: functionName}, `log`.
- Placeholders in args: `{value:id.prop}`, `{value:self.prop}`, `{var:name}`, `{data:field}`.
- A script prints ONE JSON object of JSON actions on stdout. Keys are applied in Alfred's fixed order:
  `remove, insert, update, replace, sheet, share, sound, speak, copy, open, opendialog, window,
  notification, functions, popupmenu, finish`, plus `variables` and `pulse`.
  - `update`: `{ "ids": { "items": { "data": { "src": [rows] } }, "statusLabel": { "text": "…" } },
    "relayout": true }` (relayout when text gets longer).
  - `functions`: `{ "ids": { "items": ["filterdata", "selectfirst"] } }`.
  - `popupmenu`: `{ "src": "anchorButtonId", "menu": [{ "text": "Open\t↩", "action": {…} },
    { "type": "separator" }, { "text": "Open in", "submenu": [...] }] }` → the ⌘K menu.
  - `sheet`: `{ "id": "s", "closeonescape": true, "root": {…} }`, close with `{ "sheet": { "close": "s" } }`.
  - `opendialog`: `{ "canchoosefiles": false, "canchoosefolders": true, "setvariable": "picked",
    "action": [...] }` then read `{var:picked}`.
  - `copy` {type: text, src}, `open` {type: url|file, src}, `sound` {src}, `notification`.
  - `finish`: `{ "args": "…", "variables": {…} }` closes the window and continues the workflow.
- Variables: `variablesconfig: { name: { default, persist, scriptenv, src: "args[0]" | "variable" } }`.
  `scriptenv: true` passes it to scripts as env; `persist` remembers it between launches (needs
  `bundleid`); `src: "args[0]"` takes the query the window was opened with; `src: "variable"` takes a
  workflow variable.
- `pulse` {src, args, interval, intervalmode}: run a script on a timer (e.g. TOTP countdown).

## Proven patterns

- **Search window**: top `view` (border) with a ‹ back button (left), a borderless big `textfield`
  (centre), optional `picker` (right); centre `dataview`; bottom footer `view` with a status label
  (left) and hint buttons (right: "Open ↩ | Actions ⌘K").
- **Load on `launched`**, never `preload` (see gotchas). Send pickers/labels as separate script
  actions from the rows.
- **⌘K**: a footer button `actionsButton` whose action runs a script that returns `popupmenu`
  anchored on it; the root `shortcuts` maps `"⌘K"` (configurable) to the same script.
- **Modifier shortcuts** (⇧⌘F, ⌘↩, ⌘Y): zero-size, `opacity: 0` footer buttons with `shortcut`.
- **Back to the menu**: `finish` with `variables: { out: "back" }` → a Run Script after the window
  re-opens the Script Filter via AppleScript `tell application id "com.runningwithcrayons.Alfred" to
  search "kw "`. ⎋ can't do it: Alfred closes windows on ⎋ itself.
- **Long work** (install, clone): `[json(update status "Installing…"), script(..., async: true)]`.
- **Preview**: a `sheet` with a `webkitview` (`text` = your own HTML) sized on the webkitview itself.
- **Live preview with undo**: write the change on `changed`, remember the original in a cache file,
  restore it on the root view's `quit` event unless ↩ committed it.
