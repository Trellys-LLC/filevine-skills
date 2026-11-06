# Filevine Skills

Free, open Agent Skills for working with [Filevine](https://www.filevine.com) from Claude. Made by Trellys.

## Skills

| Skill | What it does |
|---|---|
| `filevine` | Connect Claude to Filevine's official connector, know what it can and cannot reach, read data safely, and build live dashboards. |

## Install

- **Claude desktop / claude.ai:** zip the `skills/filevine` folder and upload it under Customize > Skills.
- **Claude Code:** copy `skills/filevine` into `~/.claude/skills/`.

Connect Filevine first, from Customize > Connectors in claude.ai. Claude Code cannot add Filevine by URL (Filevine's sign-in server does not allow dynamic client registration), but connectors from your claude.ai account appear there.

## Notes

- The skills contain instructions only. No client data, no credentials.
- Filevine changes its connector over time. Each skill says when it was last checked.
- Not affiliated with or endorsed by Filevine. Not legal advice.

License: Apache-2.0.
