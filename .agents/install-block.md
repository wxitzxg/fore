# The canonical install block

One install story, one wording. `README.md` and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate.

fore ships through the `wxitzxg` marketplace, which is this repo's own `.claude-plugin/marketplace.json`. Adding the marketplace registers the plugin; installing the plugin enables the skills.

## Claude Code: the plugin

<canonical-block name="claude-code">

Add the marketplace once:

```
/plugin marketplace add wxitzxg/fore
```

Then install:

```
/plugin install fore@wxitzxg
```

Or from the shell:

```bash
claude plugin marketplace add wxitzxg/fore
claude plugin install fore@wxitzxg
```

</canonical-block>

## Codex, and other agents: skills.sh

The plugin is Claude Code only. Everywhere else, skills.sh copies editable skill files into the project. Use the whole-set form on `README.md`:

<canonical-block name="skills-sh-whole-set">

```bash
npx skills@latest add wxitzxg/fore
```

Pick the skills you want, and which coding agents to install them on. **The installer lets you choose which skills to take: make sure `setup` is one of them.**

</canonical-block>

…and the single-skill form wherever one skill is named on its own.

<canonical-block name="skills-sh-one-skill">

```bash
npx skills@latest add wxitzxg/fore --skill=<name>
```

```bash
npx skills@latest update <name>
```

</canonical-block>

`skills@latest` is the pinned spelling in all three. `docs/` pages do not repeat the commands inline: link to `README.md` instead of duplicating them.

## The two routes are exclusive

The plugin is a managed, read-only bundle you subscribe to. skills.sh writes files you own and edit. Installing both leaves the user with every skill twice: always say "pick one".
