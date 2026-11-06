# Filevine Skills

Free, open Agent Skills for working with [Filevine](https://www.filevine.com) from Claude. Made by Trellys.

## Skills

| Skill | What it does |
|---|---|
| `filevine` | Connect Claude to Filevine's official connector, know what it can and cannot reach, read data safely, and build live dashboards. |

## Install

**Easiest:** open Claude and say:

> Install the Filevine skill from https://github.com/Trellys-LLC/filevine-skills

Claude fetches it and sets it up. Then connect Filevine (next section).

**By hand in Claude desktop or claude.ai:** download this repository (green Code button, Download ZIP), unzip it, and zip just the `skills/filevine` folder (the one that contains `SKILL.md`). Upload that zip under Customize > Skills. If Claude asks for a `.md` file, the file it wants is `SKILL.md` inside that folder, not the whole download.

**Claude Code:** copy `skills/filevine` into `~/.claude/skills/`.

## Connect Filevine first

In claude.ai: Customize > Connectors > add Filevine > sign in. Claude Code cannot add Filevine by URL (Filevine's sign-in server does not allow dynamic client registration), but connectors from your claude.ai account appear there.

## What to say once it is installed

- "Learn how my Filevine is set up." Claude reads your project types, phases and sections and saves what it learns in one Claude page called Filevine Models, so later chats start faster. You can share that page with your team.
- "Build me a dashboard of open cases by phase." Live dashboards read from Filevine each time they open.
- "What can Claude and can't Claude get from Filevine?"

Every firm sets Filevine up differently, so the first run takes a little while. It learns more the more you use it.

## Notes

- The skills contain instructions only. No client data, no credentials.
- Filevine changes its connector over time. Each skill says when it was last checked.
- Not affiliated with or endorsed by Filevine. Not legal advice.

License: Apache-2.0.
