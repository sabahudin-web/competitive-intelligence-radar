---
description: The weekly competitor gather run. Fans out one competitor-scout subagent per competitor in parallel, each returning a cited dossier via BrightData, then diffs against last week, merges one briefing, and publishes to all three surfaces (local folder, Notion, HTML dashboard). Runs autonomously. Run /radar-setup first.
---

# /radar

You are the gather orchestrator. This run is AUTONOMOUS end to end. You use SUBAGENTS for gathering,
never an agent team, because scouts work independently and only report back. See
[subagents-vs-agent-teams.md](../rules/subagents-vs-agent-teams.md).

Precondition: `radar.config.json` exists with verified MCP prefixes. If not, stop and tell the user
to run `/radar-setup`.

## Sequence

1. Run the `radar-plan-targets` skill to read the config, enforce the cap of 8 and waves of 8, create
   the run dossier directory, and produce one scout job spec per competitor.

2. Fan out the scouts. Spawn one `competitor-scout` subagent PER competitor, all in ONE message
   (multiple tool calls in a single response) so they run in parallel. Give each scout its job spec
   from step 1: the competitor, my keywords, geo, scout depth flags, the live BrightData prefix, the
   freshness windows, and its unique `OUTPUT_PATH`. Scouts gather with BrightData, write one cited
   JSON dossier each, and report back. They never message each other. If there are more than 8
   competitors (there should not be after the cap), run a second wave after the first returns.

3. Run the `radar-diff-changes` skill to compare this run's dossiers against last run and produce a
   ranked, freshness-aware change set.

4. Run the `radar-merge-briefing` skill to write the three local artifacts: briefing markdown, the
   combined raw dossier JSON, and the dashboard data JSON. Every claim carries a source URL.

5. Run the `radar-publish-notion` skill to upsert the Competitors rows (with the Change field) and
   create the Briefings page. If Notion is not provisioned, note it and continue; the local and
   dashboard surfaces still publish.

6. Run the `radar-render-dashboard` skill to inject the data into the self-contained HTML dashboard.

7. Print the THREE output locations clearly:
   - Local briefing markdown path and combined dossier JSON path.
   - Notion briefing page link (or a note that Notion was skipped).
   - Local dashboard HTML path (offer to open it).
   Then hint: "Run `/radar-council` to convene the strategic war room over this briefing."

## Rules in force

- No fabrication, freshness windows, cost control (batch tools, caps), no em or en dashes. See the
  files in `rules/`.
