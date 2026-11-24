---
name: zsk-alfred-workflow
description: >
  Build, fix, review or publish Alfred 5/6 workflows, especially Alfred 6 workflows with Alfred UI
  windows (ui.json). Loads the official doc links, the Alfred UI API, the project layout (Bun +
  TypeScript, ui.json and info.plist generated from code), hard-won Alfred UI gotchas, and the
  sharing rules (forum tips, Gallery guidelines, Style Guide, AI disclosure). Triggers on
  /zsk-alfred-workflow, "alfred workflow", "alfred ui", "alfredui", "ui.json", ".alfredworkflow".
---

# zsk-alfred-workflow

Guidelines for building Alfred workflows that feel native to Alfred, work on other people's Macs, and
can be shared with the community. Read the reference files only when the task needs them.

| File | Read when |
| --- | --- |
| [references/docs.md](references/docs.md) | You need an official doc page (always prefer it over memory: Alfred 6 is new). |
| [references/alfred-ui.md](references/alfred-ui.md) | Building or changing an Alfred UI window (`*.alfredui/ui.json`). |
| [references/gotchas.md](references/gotchas.md) | **Before writing any Alfred UI code**, and whenever something silently does nothing. |
| [references/project.md](references/project.md) | Starting a new workflow project, adding objects to info.plist, testing, packaging, releasing. |
| [references/sharing.md](references/sharing.md) | Writing the README / workflow readme, screenshots, releasing, forum post, Gallery submission. |

## How to work

1. **Check the docs first.** Alfred UI shipped with Alfred 6 (late 2026): your training data is thin or
   wrong. Fetch the page from `references/docs.md` for any component, event or JSON action you use.
   If a property isn't documented, say so and test it rather than guess.
2. **Native first.** Prefer Alfred's own objects (Keyword, Script Filter, File Filter, Universal Action,
   File Action, Hotkey, Configuration Builder) and only use an Alfred UI window when a list needs
   things Alfred's result list can't do (pickers, pills, previews, sheets, ⌘K menus). A native
   Script Filter is one ↩; a window adds a step.
3. **Read `gotchas.md` before coding.** Most Alfred UI failures are silent (the whole action is dropped).
4. **Generate, don't hand-edit.** Windows are TypeScript objects compiled to `ui.json`; `info.plist` is
   generated from `scripts/plist.ts`. See `project.md`.
5. **Test like Alfred runs it:** from the window folder with a bare environment
   (`env -i HOME="$HOME" PATH=/usr/bin:/bin ../bin/<cli> <command>`), and check the JSON it prints.
   Then reload the workflow and have the user check the window on screen.
6. **Ship clean:** configurable keywords (3+ characters), no hardcoded paths, no personal data in
   screenshots or packages, AI disclosure, credits for anything adapted. See `sharing.md`.

## Rules

- Paths in the workflow are relative to the workflow folder (`./bin/...`, `../bin/...` from a window).
- Every user setting goes in the **Configuration Builder** (`userconfigurationconfig`), never as an
  edit to the workflow. `prefs.plist` holds the user's choices: git-ignore it and never package it.
- Keywords come from configuration variables (`{var:kw_main}`), default to 3+ characters.
- Never auto-update, never install software at runtime (`brew`, `pip`, `curl` a binary): the Gallery
  rejects it. Bundled binaries must be signed and notarised; scripts are fine.
- Scripts in Alfred get a minimal `PATH`: use absolute paths (`/usr/bin/git`, `/usr/bin/sqlite3`,
  `/usr/bin/osascript`) or search the usual install locations (see the launcher in `project.md`).
- Secrets go through stdin or env, never argv. Don't log them.
- Disclose AI help in the README, the workflow readme and the forum post (wording in `sharing.md`).
- Before saying something works, run it (bare env) and say what was and wasn't seen on screen.
