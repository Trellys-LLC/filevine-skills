# Filevine Skills

Free, open skills for working with [Filevine](https://www.filevine.com) from Claude. Made by Trellys.

## Install (about a minute)

1. **Download [filevine-plugin.zip](https://github.com/Trellys-LLC/filevine-skills/raw/main/filevine-plugin.zip).** Do not unzip it.
2. In Claude, open **Customize > Plugins**, choose **Add > Upload plugin**, and upload that zip.
3. **Connect Filevine:** Customize > Connectors > Filevine > sign in.

That's it. Open a new chat and try one of these:

- "Learn how my Filevine is set up."
- "Build me a dashboard of open cases by phase."
- "What can Claude get from Filevine, and what can't it?"

Every firm sets up Filevine differently, so the first run takes a little while. Claude saves what it learns in one page called Filevine Models, so later chats start faster and you can share the page with your team.

**Claude Code (plugin):** run `/plugin marketplace add Trellys-LLC/filevine-skills`, then `/plugin install filevine@trellys-skills`. Turn on auto-update for the marketplace under `/plugin` > Marketplaces if you want new versions without asking.

**Claude Code (without the plugin):** say "Install the Filevine skill from https://github.com/Trellys-LLC/filevine-skills", or copy the three folders in `skills/` into `~/.claude/skills/`. Claude Code cannot add Filevine by URL, but a Filevine connector from your claude.ai account appears there.

## Notes

- The plugin has three skills: `filevine` (connect, what Filevine cannot give, reading data safely), `filevine-dashboards`, and `filevine-firm-model` (finding out and remembering how a firm has set up Filevine). They are instructions only. No client data, no credentials.
- Filevine changes its connector over time. The skill says when it was last checked.
- `filevine-plugin.zip` is built from `.claude-plugin/plugin.json`, `skills/` and `LICENSE`. Rebuild it whenever any of them changes, and raise `version` in `plugin.json` on each release so Claude Code users receive it.
- Not affiliated with or endorsed by Filevine. Not legal advice.

License: Apache-2.0.
