# Reference: radar.config.json schema

`radar.config.json` holds the user's own business context and run state. It lives at the project root
and is GITIGNORED, so the shared plugin stays generic and no user data ever reaches the repo.
`radar-onboard` writes it; other skills read it.

## Full shape

```json
{
  "version": 1,
  "business": {
    "name": "Acme Co",
    "domain": "acme.com",
    "one_liner": "What the business does, in one plain sentence.",
    "geo": "us"
  },
  "keywords": [
    "primary category keyword",
    "secondary keyword",
    "buyer intent phrase"
  ],
  "competitors": [
    { "name": "Competitor One", "domain": "competitor-one.com" },
    { "name": "Competitor Two", "domain": "competitor-two.com" }
  ],
  "scout_depth": {
    "social": false,
    "financials": false,
    "hiring": false
  },
  "freshness_overrides": {
    "news_days": 30,
    "funding_days": 90,
    "sentiment_days": 30
  },
  "mcp": {
    "brightdata_prefix": "mcp__claude_ai_Bright-Data__",
    "notion_prefix": "mcp__claude_ai_Notion__",
    "brightdata_verified_at": "2026-06-16T14:00:00Z",
    "notion_verified_at": "2026-06-16T14:00:00Z"
  },
  "notion": {
    "parent_page_id": "",
    "competitors_db": "",
    "competitors_ds": "",
    "briefings_db": "",
    "briefings_ds": ""
  },
  "last_run": {
    "date": "",
    "briefing_path": "",
    "dossier_dir": "",
    "dashboard_path": "",
    "briefing_notion_page_id": ""
  }
}
```

## Field notes

- `business`, `keywords`, `competitors` are collected interactively at onboarding. `geo` is a
  2-letter country code passed to BrightData `geo_location`.
- `competitors` is capped at 8 by [cost-control.md](../rules/cost-control.md). Extra entries are
  ignored with a warning.
- `scout_depth` flags turn on the opt-in deep BrightData sources. All default to `false` for a cheap
  lean weekly run. See [brightdata-tool-map.md](brightdata-tool-map.md).
- `mcp.*_prefix` is detected live by `radar-onboard` (could be the plugin prefix, the desktop
  `claude_ai` prefix, or a UUID prefix). Every other skill builds tool names from these.
- `notion.*` is filled by `radar-provision-notion` after the user approves the schema. Pages are
  created under the `*_ds` (data source) ids.
- `last_run` lets `radar-diff-changes` compare this run to the previous one and lets
  `radar-publish-council` find the briefing to append to.

## Edge cases

- Missing file: any radar skill that needs it stops and tells the user to run `/radar-setup` first.
- Partial file (for example onboarding done but Notion not provisioned): skills check the specific
  keys they need and route the user to the right setup step rather than failing opaquely.
