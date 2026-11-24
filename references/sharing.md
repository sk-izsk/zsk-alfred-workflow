# Sharing: forum, Gallery, Style Guide, disclosure

Sources (re-check them before a release, they change): forum READ FIRST tips, https://alfred.app/submit/,
https://alfred.app/submit/styleguide/, https://alfred.app/submit/screenshots/ (links in `docs.md`).

## Forum "READ FIRST" tips (Share your Workflows)

- It works smoothly: **relative paths** (nothing hardcoded to your Mac), keywords useful to others,
  every action tested. Half-done workflows go to Workflow Help & Questions instead.
- Use the **Configuration Builder** for preferences (keywords, paths, API keys, options).
- Clear documentation: what it does, main keywords, anything to configure first (Style Guide).
- **Disclose AI/LLM help** in the documentation **and** the forum post.

## Gallery (https://alfred.app/submit/)

- Invitation only: share on the forum first; the team may invite stable, trusted workflows.
- Must respect privacy and be safe; the community must be able to audit the contents.
- **Never self-update.** Never download or install software after install (`brew`/`pip`/`gem`
  install, curl a binary). Bundling libraries is fine; Homebrew dependencies are declared, Alfred
  handles them.
- Compiled binaries must be **signed and notarised** (scripts aren't binaries).
- Keywords of **at least 3 characters**, ideally user-configurable. Main icon **at least 256×256**.
- Strong candidates allow user configuration and follow the Style Guide.

## Style Guide (README and the workflow's About text)

- Start with `## Usage`, one or two short sentences per action, using "Search" for filtering, ending
  "via the `keyword` keyword" (also for Script Filters) or "via the Universal Action".
- Modifiers as a list: `* <kbd>⌘</kbd><kbd>↩</kbd> Open in Terminal` (one `<kbd>` per key).
- Alternative ways to do the same task: "Alternatively, …" plus a second screenshot.
- Hotkeys: "Configure the Hotkey to …". Configuration: say "Workflow's Configuration"; explain
  settings in the configuration itself and only mention easily-missed ones in the readme.
- `## Setup` above Usage only for extra manual steps (not required config, apps, formulae or API keys:
  the Gallery lists those).
- Screenshots: `![alt](images/name.png)`, **only the Alfred window** (⌘⇧4, Space, click: transparent
  shadow), only this workflow's results, realistic data, no other workflows.

## Privacy checklist before publishing

- Screenshots use demo data: no real project names, paths, emails, profile names, other windows.
  Check every image yourself. If a personal screenshot was committed, it stays in git history: squash
  before merging, or rewrite the branch (may need force-push rights).
- `unzip -l` the `.alfredworkflow`: no `prefs.plist`, `.DS_Store`, tool folders (`.omc`, `.claude`),
  caches, logs or absolute paths.
- No secrets in the repo, the package, logs or argv.

## AI disclosure wording

README (`## AI disclosure`):
> This workflow was built with the help of AI tools, directed and tested by a human. Review the code
> before use, as you would any workflow; please report anything that doesn't work.

Workflow readme (info.plist, last line):
> This workflow was built with the help of AI tools. Review it before use, and report issues on the project page.

Keep it tool-neutral unless the owner wants the tool named. Same sentence in the forum post.

## Credits

Adapting another project (MIT etc.): name it and link it in the README `## Credits`, keep its licence
notice in `THIRD_PARTY_NOTICES.md`, ship that file inside the `.alfredworkflow`. Use only original
icons (no product logos). Verify author names and URLs from the source, never from memory.

## Release checklist

1. `version` bumped; `bun run check` green; workflow reloaded and smoke-tested.
2. `bun run package`; inspect the archive.
3. Merge to the default branch; `gh release create vX.Y.Z <Name-X.Y.Z>.alfredworkflow` with notes
   (requirements, what it does, editors/apps tested, credits).
4. Forum post in Share your Workflows (screenshots, keywords, setup, AI disclosure, link).
