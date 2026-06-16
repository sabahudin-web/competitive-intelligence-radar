---
name: radar-onboard
description: Run this FIRST, before any other radar skill. Sets up Competitive Intelligence Radar by ASKING whether BrightData and Notion are connected, live-testing one real call to each, detecting which MCP prefix is actually live, and writing radar.config.json with the user's business context. Use when the user runs /radar-setup, says "set up the radar", "onboard", or when any radar skill reports that radar.config.json is missing. Does not assume tools are connected; it verifies.
---

# radar-onboard

Automation level: CHECKPOINT. You ask the user real questions and you stop if a live MCP test fails.
Never assume a connection. See [automation-levels.md](../../rules/automation-levels.md).

Goal: produce a complete `radar.config.json` (schema in
[config-schema.md](../../references/config-schema.md)) with verified MCP prefixes, so every later
skill just works.

## Steps

1. Greet and explain in two lines what the radar does and that it needs BrightData (web data) and
   Notion (publishing). Keep branding generic; this plugin serves any business.

2. ASK about connections, do not assume:
   "Have you connected BrightData and Notion? I usually see two paths: through this plugin, or
   through the Claude Desktop app connectors. I will test both."

3. Detect the live BrightData prefix. Try a tiny real call in this order until one works, using a
   harmless query:
   - Plugin path: `mcp__plugin_competitive-intelligence-radar_brightdata__search_engine`
   - Desktop path: `mcp__claude_ai_Bright-Data__search_engine`
   - Any other connected BrightData server visible in this session (a UUID prefix is normal).
   Use query like `{"query": "site:example.com", "engine": "google"}`. The FIRST prefix that returns
   a result is the live one. Record `mcp.brightdata_prefix` and `mcp.brightdata_verified_at`.

4. Detect the live Notion prefix. Try a tiny real read, for example `notion-get-users` or a
   `notion-search` with a simple query, in this order:
   - `mcp__plugin_competitive-intelligence-radar_notion__`
   - `mcp__claude_ai_Notion__`
   - Any other connected Notion server (UUID prefix).
   Record `mcp.notion_prefix` and `mcp.notion_verified_at`.

5. If EITHER test fails, STOP. Do not write a half-config that pretends a connection exists. Tell the
   user exactly how to connect:
   - BrightData: get a free token at brightdata.com, then either set `BRIGHTDATA_API_TOKEN` in the
     environment so this plugin's `.mcp.json` server starts, or add BrightData in the Claude Desktop
     app connectors. Re-run `/radar-setup` after.
   - Notion: connect Notion via OAuth, either through this plugin's Notion server or the Desktop app
     connectors. Re-run after.

6. Collect business context interactively and confirm it back:
   - Business name, domain, one-line description, 2-letter geo.
   - Tracked keywords (the searches where ranking matters).
   - Competitor list: name + domain. Remind the user of the cap of 8
     ([cost-control.md](../../rules/cost-control.md)); if they give more, keep the first 8 and say so.
   - Scout depth: ask whether to include social sentiment, financials, hiring. Default all to off for
     a cheap lean run; the user can turn them on.

7. Ask where in Notion to put the databases: a parent page id or "create at workspace level."
   Store as `notion.parent_page_id` (may be empty for workspace level). Do NOT create databases here;
   that is `radar-provision-notion`'s job after the user approves the schema.

8. Write `radar.config.json` at the project root with everything gathered and the two verified
   prefixes. Confirm the file path and summarize what was saved (no secrets).

9. Tell the user the next step: run `/radar-setup` continues into `radar-provision-notion` to review
   and approve the Notion schema, then `/radar` for the first gather.

## Edge cases

- Both prefixes live but pointing at different connection methods (for example BrightData via plugin,
  Notion via desktop): that is fine. Record each independently.
- A prefix returns an auth error rather than a clean result: treat as a failed test, stop, and give
  the connect instructions for that service.
- Re-running onboarding: load the existing config if present, prefill answers, and let the user edit
  rather than starting blank. Overwrite only after confirmation.
- User refuses a deep source: leave its `scout_depth` flag false; never enable a source the user did
  not ask for.
