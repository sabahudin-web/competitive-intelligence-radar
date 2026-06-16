---
description: One-time setup for Competitive Intelligence Radar. Asks about your BrightData and Notion connection and live-tests both, then shows the proposed Notion schema for your approval before creating anything, then checks the agent-teams prerequisites for the council. Run this before /radar.
---

# /radar-setup

You are orchestrating setup. This command is CHECKPOINT throughout: it asks before it acts. It calls
skills in order and stops on any failure. Do the work through the skills; this file only sequences
them. See [automation-levels.md](../rules/automation-levels.md).

## Sequence

1. Run the `radar-onboard` skill. It ASKS whether BrightData and Notion are connected, live-tests one
   real call to each, detects the live MCP prefix for each (plugin path, desktop `claude_ai` path, or
   a UUID prefix), and writes `radar.config.json` with the user's business context. If either live
   test fails, it STOPS with connection instructions. Do not continue past a failed test.

2. Run the `radar-provision-notion` skill. It SHOWS the proposed Competitors and Briefings schema and
   asks "do you like this, or want changes?" It creates the databases ONLY after the user approves,
   and is idempotent if they already exist. Store the database and data source ids in the config.

3. Check the agent-teams prerequisites for `/radar-council` and report status to the user (do not try
   to fix their settings silently):
   - Claude Code version is v2.1.32 or later (`claude --version`).
   - `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` is set to `1` in settings or environment.
   If either is missing, show the exact fix from
   [agent-teams-setup.md](../references/agent-teams-setup.md). Gather and the council are independent:
   the user can run `/radar` even if the council prerequisites are not met yet.

4. Summarize: what was verified, what was created in Notion, whether the council is ready, and that
   the next step is `/radar`.
