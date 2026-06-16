# CLAUDE.md

Guidance for Claude Code when working in the Competitive Intelligence Radar plugin repository.

## What this project is

A Claude Code plugin that runs a weekly competitor radar for business owners and convenes a strategic
war room on demand. It showcases the BrightData MCP for live web data and deliberately uses BOTH
Claude Code parallelism primitives, each where it honestly fits.

- `/radar-setup` onboards: asks about and live-tests the user's BrightData and Notion connection,
  shows the proposed Notion schema for approval, checks agent-team prerequisites.
- `/radar` is the weekly GATHER: it fans out one `competitor-scout` SUBAGENT per competitor in
  parallel; each gathers a cited dossier and reports back; the orchestrator diffs vs last run, merges
  one briefing, and publishes to three surfaces.
- `/radar-council` is the on-demand ANALYZE: it spins up a real experimental AGENT TEAM of four
  analyst teammates who message and challenge each other, then publishes a Strategic War Room
  addendum.

## First thing to do in a fresh checkout

If the user has just installed or cloned this, point them to `/radar-setup`. Nothing else runs until
`radar.config.json` exists with verified MCP prefixes.

## Architecture you must respect

- GATHER uses SUBAGENTS. Scouts are independent and only report back; they never message each other.
  Do not turn the gather into an agent team.
- DEBATE uses an AGENT TEAM. Analysts must see and challenge each other, which only a team allows.
  Do not turn the debate into plain subagents.
- The full rationale is in `rules/subagents-vs-agent-teams.md`. Read it before changing either flow.

## Hard rules (these are not optional)

These live in `rules/`. Every generated artifact must obey them:

1. `no-fabrication.md`: every factual claim carries a real `source_url` fetched this run plus
   `fetched_at`. No value without a source. Model knowledge is never a source.
2. `freshness.md`: every field has a recency window and a `fetched_at`; stale data is labeled, not
   presented as current.
3. `cost-control.md`: prefer `*_batch` BrightData tools; cap competitors at 8; lean scout depth by
   default; the council is token heavy.
4. `automation-levels.md`: AUTONOMOUS by default; CHECKPOINT for Notion database creation, for the
   MCP connection check at onboarding, for `/radar-council`, and for any GitHub push.
5. `no-em-dashes.md`: never use the em dash (U+2014) or en dash (U+2013) anywhere, including in this
   file. Use commas, colons, or hyphens.

## MCP tool resolution (important)

The same server resolves under different prefixes depending on connection method:

- Plugin path: `mcp__plugin_competitive-intelligence-radar_<server>__<tool>`
- Desktop app connectors: `mcp__claude_ai_<Server>__<tool>`
- Some sessions: a UUID prefix `mcp__<uuid>__<tool>`

Always refer to BrightData and Notion tools BY CAPABILITY and build the full name from the prefix that
`radar-onboard` detected and stored in `radar.config.json` (`mcp.brightdata_prefix`,
`mcp.notion_prefix`). See `references/brightdata-tool-map.md`.

The agent-team caveat: when an `agents/*.md` definition runs as a TEAMMATE, its `skills` and
`mcpServers` frontmatter are NOT applied. Teammates load MCP and skills from the USER's settings. This
is why onboarding verifies the user's own connection. See `references/agent-teams-setup.md`.

## Layout

```
.claude-plugin/   plugin.json + marketplace.json (repo is its own marketplace, source "./")
.mcp.json         brightdata + notion, bare server-name keys (no mcpServers wrapper)
commands/         /radar-setup, /radar, /radar-council (orchestrate only)
agents/           competitor-scout (subagent, sonnet) + 4 analyst teammate roles (sonnet)
skills/           radar-onboard, -provision-notion, -plan-targets, -diff-changes,
                  -merge-briefing, -publish-notion, -render-dashboard, -publish-council
rules/            the 6 hard rules above
references/        notion-schema, brightdata-tool-map, agent-teams-setup, config-schema
assets/           briefing-template.md, dashboard-template.html (self-contained, no deps)
outputs/          briefings/, dossiers/, dashboard/ (generated, gitignored)
radar.config.json the user's business context and run state (gitignored, never committed)
```

## Conventions when editing

- Reference bundled files from commands and skills with `${CLAUDE_PLUGIN_ROOT}`.
- Component directories live at the plugin ROOT, not under `.claude/`.
- The dashboard must stay self-contained: inline CSS and JS, data injected as inline JSON at the
  `{{RADAR_DATA_JSON}}` placeholder, no external requests, no build step.
- Keep all user-specific data in `radar.config.json` so the shared plugin stays generic.
- Skills are the workers; commands only sequence skills and (for `/radar`) fan out subagents.

## Verifying a change

- `node -e "JSON.parse(...)"` the three JSON config files.
- Grep the repo for U+2014 and U+2013; there should be zero hits.
- Confirm every dossier field is shaped `{ value, source_url, fetched_at, status }`.
- For flow changes, dry-run the relevant command and confirm all three surfaces still publish.
