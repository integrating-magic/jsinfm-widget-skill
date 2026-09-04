# jsinfm-widget — Claude skill for FileMaker web-viewer widgets

A [Claude skill](https://docs.claude.com) that scaffolds and builds self-contained JavaScript or React widgets that run inside a FileMaker web viewer — charts, tables, formatted displays, or full React + TanStack Query widgets that fetch live FileMaker data via FMGofer and send events back with `WV Event`.

Maintained by [Integrating Magic](https://integratingmagic.io) for the JS in FM community.

## Install

The skill is a single folder, `jsinfm-widget/`, containing `SKILL.md`.

**Claude Code (CLI)**

```bash
# personal skills (all projects)
git clone https://github.com/integrating-magic/jsinfm-widget-skill.git /tmp/jsinfm-widget-skill
cp -r /tmp/jsinfm-widget-skill/jsinfm-widget ~/.claude/skills/

# or per-project
cp -r /tmp/jsinfm-widget-skill/jsinfm-widget .claude/skills/
```

Then in Claude Code: `/jsinfm-widget` (or just ask to build a FileMaker web-viewer widget).

**Claude desktop app / Cowork / claude.ai**

Go to Settings → Skills (Capabilities), add a skill, and upload the `jsinfm-widget` folder (or `SKILL.md`). It then syncs to all of your Claude surfaces.

## Update

```bash
cd /tmp/jsinfm-widget-skill && git pull
cp -r jsinfm-widget ~/.claude/skills/
```

Changes are listed in [CHANGELOG.md](CHANGELOG.md). Releases are tagged (`v1.0.0`, …).

## What it does

When invoked, the skill asks which of three builds you want and routes accordingly:

1. **Simple widget** — a small self-contained chart/table/display, usually vanilla JS with a library such as ApexCharts or DataTables.
2. **Practice / learning** — a lightweight environment for practicing JavaScript inside FileMaker; you write the JS.
3. **Complex, data-fetching widget** — the full React + TanStack Query scaffold that fetches live FileMaker data through the `Fetch Data` script and sends events back through `WV Event`.

Both dev environments it uses are open source:

- Vanilla JS: https://github.com/integrating-magic/js-dev-environment-new
- React + TanStack Query: see `SKILL.md`

## Contributing

Open an issue or PR. Keep `SKILL.md`'s frontmatter `name` as `jsinfm-widget` — that's what Claude uses to identify and trigger it.

## License

MIT — see [LICENSE](LICENSE).
