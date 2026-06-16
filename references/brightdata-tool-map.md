# Reference: BrightData Tool Map (capability to tool)

Scouts and analysts refer to BrightData tools BY CAPABILITY, never by a hardcoded full name, because
the same server resolves under different prefixes depending on how it is connected:

- Connected through this plugin's `.mcp.json`:
  `mcp__plugin_competitive-intelligence-radar_brightdata__<tool>`
- Connected through the Claude Desktop app connectors:
  `mcp__claude_ai_Bright-Data__<tool>` (the exact server label can vary)
- In some sessions the server appears under a UUID:
  `mcp__<uuid>__<tool>`

`radar-onboard` live-detects which prefix actually answers and records it in
`radar.config.json` as `mcp.brightdata_prefix`. Everywhere else, use the recorded prefix + the tool
name from the right column.

## Core sources (always run; lean default)

| Capability | Tool | Notes |
| --- | --- | --- |
| SERP rank for my keywords | `search_engine_batch` | Batch all tracked keywords in one call. `engine` google, optional `geo_location`. Returns JSON for Google. Find the competitor's best position. |
| One-off SERP / news lookup | `search_engine` | Single query fallback and news discovery. |
| Scrape home / pricing / product | `scrape_batch` | Up to 10 URLs per call, returns Markdown. Unlocks bot-protected pages. Prefer over single scrapes. |
| Single page scrape | `scrape_as_markdown` | Use only when one URL is needed. |
| News (wire) | `web_data_reuter_news` | Structured recent news. Pair with `search_engine` for broader coverage. |

## Deep sources (opt-in via radar.config.json scout_depth flags)

| Capability | Tool | Flag |
| --- | --- | --- |
| Sentiment on X | `web_data_x_posts` | `scout_depth.social` |
| Sentiment on Reddit | `web_data_reddit_posts` | `scout_depth.social` |
| Company reviews | `web_data_facebook_company_reviews` | `scout_depth.social` |
| Financials | `web_data_yahoo_finance_business` | `scout_depth.financials` |
| Hiring direction | `web_data_linkedin_job_listings` | `scout_depth.hiring` |
| Company profile (optional context) | `web_data_linkedin_company_profile` | `scout_depth.hiring` |
| Funding context (optional) | `web_data_crunchbase_company` | `scout_depth.financials` |

## Fallback only (most expensive, use when blocked or JS heavy)

| Capability | Tool |
| --- | --- |
| Open a page in a real browser | `scraping_browser_navigate` |
| Read rendered text | `scraping_browser_get_text` |
| Read rendered HTML | `scraping_browser_get_html` |

Reach for the scraping browser ONLY when `scrape_batch` returns a block, a CAPTCHA, or empty
JavaScript-rendered content for a specific URL. See [cost-control.md](../rules/cost-control.md).

## Cost rules that apply here

1. Always prefer `*_batch` over per-URL calls.
2. Run deep sources only when their flag is on.
3. Every value returned must be wrapped with `source_url` + `fetched_at`, per
   [no-fabrication.md](../rules/no-fabrication.md).
