# zsk-alfred-workflow

A Claude Code skill for building, fixing and publishing **Alfred workflows**, especially Alfred 6
workflows with **Alfred UI** windows. Invoke it with `/zsk-alfred-workflow`, and the model gets:

- **Links to the official docs** for every workflow object, the Script Filter JSON format, the
  Configuration Builder, and every Alfred UI component, event and JSON action
  (`references/docs.md`). Alfred UI is new, so the skill tells the model to fetch pages, not guess.
- **An Alfred UI cheat sheet**: window structure, dataview rows, pickers, sheets, popup menus,
  variables, and patterns that work (search windows, ⌘K menus, previews, live preview with undo).
- **Gotchas**: the silent failures we hit and how to avoid them (dropped actions, `preload` vs
  `launched`, image updates, modifier shortcuts, sheet sizing, WAL databases, Alfred's minimal PATH…).
- **A project layout**: Bun + TypeScript, windows and info.plist generated from code, tests,
  packaging and releasing.
- **Sharing rules**: the Alfred forum's READ FIRST tips, Gallery Submission Guidelines, the Style
  Guide, privacy checks, credits and AI disclosure wording.

```
SKILL.md                 when to use it, how to work, the rules
references/docs.md       official documentation links
references/alfred-ui.md  Alfred UI cheat sheet and proven patterns
references/gotchas.md    silent failures and their fixes
references/project.md    project layout, info.plist objects, testing, releasing
references/sharing.md    forum, Gallery, Style Guide, privacy, disclosure, credits
```

## Install

Copy or symlink this folder into your skills folder, for example:

```bash
ln -s "$PWD" ~/.claude/skills/zsk-alfred-workflow
```

Then type `/zsk-alfred-workflow` in Claude Code, or just ask about an Alfred workflow.

## Built from

Lessons from two open-source Alfred 6 workflows: a Bitwarden workflow and
[VS Code for Alfred](https://github.com/sk-izsk/z-vscode-workflow), plus Alfred's documentation and
Gallery guidelines (linked in `references/docs.md`). Alfred is made by Running with Crayons; this
project isn't affiliated with them.

## AI disclosure

This skill was written with the help of AI tools and reviewed by a human.

## License

MIT
