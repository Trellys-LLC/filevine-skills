# Filevine Skills

Free, open skills for working with [Filevine](https://www.filevine.com) from Claude. Made by Trellys.

## Install (about a minute)

1. **Download [filevine.zip](https://github.com/Trellys-LLC/filevine-skills/raw/main/filevine.zip).** Do not unzip it.
2. In Claude, open **Customize > Skills** and upload that zip.
3. **Connect Filevine:** Customize > Connectors > Filevine > sign in.

That's it. Open a new chat and try one of these:

- "Learn how my Filevine is set up."
- "Build me a dashboard of open cases by phase."
- "What can Claude get from Filevine, and what can't it?"

Every firm sets up Filevine differently, so the first run takes a little while. Claude saves what it learns in one page called Filevine Models, so later chats start faster and you can share the page with your team.

**Claude Code (plugin):** run `/plugin marketplace add Trellys-LLC/filevine-skills`, then `/plugin install filevine@trellys-skills`. Turn on auto-update for the marketplace under `/plugin` > Marketplaces if you want new versions without asking.

**Claude Code (without the plugin):** say "Install the Filevine skill from https://github.com/Trellys-LLC/filevine-skills", or copy `skills/filevine` into `~/.claude/skills/`. Claude Code cannot add Filevine by URL, but a Filevine connector from your claude.ai account appears there.

## Notes

- The skill is instructions only. No client data, no credentials.
- Filevine changes its connector over time. The skill says when it was last checked.
- `filevine.zip` is built from `skills/filevine`. Rebuild it whenever that folder changes.
- Not affiliated with or endorsed by Filevine. Not legal advice.

License: Apache-2.0.
