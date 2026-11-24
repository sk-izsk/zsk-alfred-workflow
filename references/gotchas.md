# Gotchas (each one cost real debugging time)

Alfred UI fails **silently**: no error, the window just doesn't change. When that happens, suspect
these first. Alfred keeps no log you can read from a script; test outputs in a bare env instead.

## Alfred UI

1. **One bad key drops the whole action.** If Alfred rejects any part of a script's JSON output, it
   ignores all of it, rows included. Keep risky updates (pickers, labels, images) in their **own**
   script action, separate from the list rows. Validate picker `selected` values against the items.
2. **Updating an image at runtime is rejected.** `update { someImage: { image: {…} } }` made the
   whole action vanish. Use static images in ui.json (`"type": "file", "src": "../icon.png"`), or set
   images through dataview row data (`"rowIcon.image.type"`, `"rowIcon.image.src"`), which works.
3. **`preload` output can be ignored.** Lists stayed empty when filled on `preload`; the same output
   worked on `launched`. Load data on `launched` (the window shows instantly, rows a moment later).
4. **Modifier shortcuts on the root view swallow other ⌘ keys.** Put ⌘↩ / ⇧⌘… on invisible footer
   buttons (`shortcut`, `opacity: 0`, size 0). Plain ⌘ letters and arrows are fine at the root.
5. **Horizontal `inline` views centre their content.** For "Title  dim detail" rows use a `border`
   view (title in `left`, detail in `centre` with `textalign: left`).
6. **A sheet sizes to its content.** Sizing the sheet's root view did nothing (tiny sheet); put the
   size on the content (e.g. the `webkitview` itself).
7. **A `webkitview` takes the keyboard once clicked**, so ⎋/↩ stop reaching the sheet: give the
   Close button `shortcut: "⎋"` and the main button `shortcut: "↩"`.
8. **No images from URLs.** Download to the workflow cache and use `type: file`; give downloads a
   time budget and `process.exit(0)` after printing so a slow download never holds the window.
9. **Unknown enum values can break the window.** Only use documented values (e.g. `linebreak`); if
   unsure, leave the property out.
10. **Workflow Configuration isn't passed to Alfred UI scripts** (Run Scripts get it as env, window
    scripts don't). Read defaults from `info.plist` (`userconfigurationconfig`) merged with the user's
    `prefs.plist`; cache the result per process.
11. **⎋ closes the window before any script runs**, so it can't mean "back". Use ⌘[ (configurable) and
    a ‹ button. Use the root view's `quit` event for cleanup (restore a previewed theme, uninstall a
    trial) and keep it fast.
12. **Every arrow press fires dataview `changed`.** Make handlers cheap and `async`, stay silent when
    nothing applies (no error sound), and debounce slow work (record the pick, sleep, re-check).

## macOS / scripts

13. **Alfred's PATH is minimal and your shell profile isn't loaded.** Find Node in Homebrew, nvm,
    Volta, fnm, `~/.local/bin`, with a "Node path" setting as fallback; call system tools by absolute
    path.
14. **`sqlite3 -readonly` fails on a WAL database that another app has open** ("unable to open
    database file"), e.g. VS Code's `state.vscdb` while VS Code runs. Open it normally and only SELECT.
15. **`git clone` can hang waiting for a password** with no TTY: set `GIT_TERMINAL_PROMPT=0`. Use
    `GIT_OPTIONAL_LOCKS=0` for `git status` so you never take a running editor's index lock.
16. **Keystroke automation** (System Events) needs Accessibility permission for Alfred; tell the user.
17. **Settings files are JSONC** (comments, trailing commas, nested blocks). Edit one top-level key
    with a small scanner; never re-serialise the whole file. Restore by removing a key you added.

## Process

18. **Generated files must be regenerated**: edit `ui/*.ts` / `scripts/plist.ts`, run the build,
    reload the workflow (`osascript -e 'tell application id "com.runningwithcrayons.Alfred" to reload
    workflow "<bundleid>"'`).
19. **You can't see Alfred.** Computer-use can't grant Alfred (background app). `screencapture -l
    <windowId>` works (Alfred windows are shareable); find the id with `CGWindowListCopyWindowInfo`.
    Region captures can include the user's other windows: crop to Alfred's windows only.
20. **Before pushing to a PR, check it's still open** (`gh pr view N --json state`); a merged PR's
    branch silently swallows new commits.
