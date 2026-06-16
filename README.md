# Competitive Intelligence Radar

A Claude Code plugin that runs a weekly competitor radar for business owners, then convenes a
strategic war room on demand. It showcases the BrightData MCP for live web data and deliberately uses
BOTH of Claude Code's parallelism features, each where it honestly fits:

- **`/radar`** fans out one scout SUBAGENT per competitor, in parallel. Each scout independently
  gathers a cited dossier (pricing, SERP rank, sentiment, news, funding, hiring) via BrightData and
  reports back. The orchestrator diffs against last week and merges one briefing where every claim
  carries a source URL.
- **`/radar-council`** spins up a real experimental AGENT TEAM of four strategic analysts who read the
  briefing, each take a lens, message and challenge each other like a scientific debate, and converge
  on recommended moves: a Strategic War Room addendum.

Subagents for the gather (workers that only report back), an agent team for the debate (teammates that
talk to each other). The plugin never swaps them. See
[rules/subagents-vs-agent-teams.md](rules/subagents-vs-agent-teams.md).

## Value

- **Save time**: one run researches every competitor at once.
- **Create focus**: the few changes that actually matter, ranked, instead of a wall of data.
- **Correct and up to date**: live web data via BrightData, with a source URL on every single claim
  and freshness windows that flag anything stale.

## Three output surfaces, every run

1. A local `outputs/` folder: the briefing markdown, the raw dossier JSON, and the war-room addendum.
2. Notion: a Competitors database (with week-over-week change tracking) and a Briefings page.
3. A self-contained local HTML dashboard you open in any browser (inline CSS and JS, no dependencies,
   no network).

## Install

This repository is its own marketplace.

```
/plugin marketplace add <your-repo-or-path>
/plugin install competitive-intelligence-radar
```

Then run `/radar-setup`.

## Connect BrightData and Notion

The plugin works whether you connect these through the plugin or through the Claude Desktop app
connectors. Onboarding live-tests both and records whichever path is active, so either works. Note
that for `/radar-council`, teammates load MCP from YOUR settings (not the plugin's frontmatter), so a
working connection in your own settings matters for the council.

### BrightData (free token)

1. Create a free account at brightdata.com and copy your API token.
2. Connect it one of two ways:
   - Through the plugin: set `BRIGHTDATA_API_TOKEN` in your environment so the server in
     [.mcp.json](.mcp.json) starts. It resolves as
     `mcp__plugin_competitive-intelligence-radar_brightdata__*`.
   - Through the Claude Desktop app: add BrightData in the connectors. It resolves as
     `mcp__claude_ai_Bright-Data__*` (or a UUID prefix in some sessions).

### Notion (OAuth)

1. Connect Notion one of two ways:
   - Through the plugin: the Notion server in [.mcp.json](.mcp.json) uses OAuth at
     `https://mcp.notion.com/sse`.
   - Through the Claude Desktop app connectors.
2. Share the parent page where you want the databases with the connected Notion integration.

If a live test fails during setup, `/radar-setup` stops and tells you exactly how to connect, then
you re-run it.

## Enable the agent team (for /radar-council)

The council uses an experimental feature. You need:

- Claude Code v2.1.32 or later (`claude --version`).
- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in your settings.json:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

`/radar` does not need this; only `/radar-council` does. Details and known limitations are in
[references/agent-teams-setup.md](references/agent-teams-setup.md).

## First run

1. `/radar-setup`: it asks about your BrightData and Notion connection and tests both, then shows the
   proposed Notion schema and asks you to approve it before creating anything, then checks the
   council prerequisites.
2. `/radar`: the weekly gather. It spawns the scout subagents in parallel and publishes the briefing
   to all three surfaces, then prints the three locations.
3. `/radar-council`: optional, on demand. It convenes the four analyst teammates over the latest
   briefing and appends the war-room addendum to all three surfaces. It asks first because it is
   token heavy.

## Weekly scheduling

To run the gather automatically each week, use the Routines panel in the Claude Desktop app: create a
routine that runs `/radar` on your chosen weekly schedule. Run `/radar-council` by hand when you want
the strategic debate, since it is token heavy and benefits from your attention.

## Getting oriented (CLAUDE.md)

This repo ships a [CLAUDE.md](CLAUDE.md) so any Claude Code session opened here understands the
architecture, the hard rules, and the conventions before it touches anything. After you install or
clone, you can run `/init` to extend or regenerate a CLAUDE.md tuned to your own setup; the shipped
one is a safe, generic starting point and contains no business data.

## Your data stays yours

Your business name, keywords, competitors, and Notion ids live only in `radar.config.json`, which is
gitignored. The shared plugin is generic; nothing about your business is committed to this repo.

## Layout

```
CLAUDE.md              orients Claude Code in this repo (architecture, rules, conventions)
.claude-plugin/        plugin.json + marketplace.json
.mcp.json              brightdata + notion servers (bare server-name keys)
commands/              /radar-setup, /radar, /radar-council
agents/                competitor-scout (subagent) + 4 analyst teammate roles
skills/                8 radar-* skills (onboard, provision, plan, diff, merge, publish, render)
rules/                 6 rules (no-fabrication, freshness, cost-control, automation-levels,
                       subagents-vs-agent-teams, no-em-dashes)
references/            notion-schema, brightdata-tool-map, agent-teams-setup, config-schema
assets/                briefing-template.md, dashboard-template.html
outputs/               briefings/, dossiers/, dashboard/ (generated, gitignored)
```

## License

MIT. See [LICENSE](LICENSE).
