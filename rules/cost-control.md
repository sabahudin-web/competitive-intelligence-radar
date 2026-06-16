# Rule: Cost Control

BrightData calls and agent-team teammates both cost money. This rule keeps a weekly run cheap and a
debate affordable, without sacrificing the citation and freshness guarantees.

## BrightData

1. Prefer batch tools. Use `search_engine_batch` and `scrape_batch` instead of issuing one
   `search_engine` or `scrape_as_markdown` call per URL. One batch call for many URLs is cheaper and
   faster.
2. Use the structured `web_data_*` collectors (for example `web_data_reuter_news`,
   `web_data_yahoo_finance_business`) rather than scraping and parsing raw HTML yourself when a
   collector exists for that source.
3. Reserve the scraping browser (`scraping_browser_navigate` + `scraping_browser_get_text`) for the
   fallback case only: a page that is blocked or JavaScript heavy. It is the most expensive path.

## Scope caps

4. Cap competitors at 8 per run. If the config lists more, process the first 8 and tell the user the
   rest were skipped.
5. Fan out scouts in waves of at most 8 parallel subagents. With the cap at 8 this is normally a
   single wave.
6. Scout depth defaults to LEAN CORE: SERP rank, homepage and pricing scrape, and news. The deeper
   sources (social sentiment, financials, hiring) run only when enabled in
   `radar.config.json` (`scout_depth.social`, `.financials`, `.hiring`). See
   [config-schema.md](../references/config-schema.md).

## Agent team (the council)

7. `/radar-council` is token heavy: each teammate is a separate Claude instance with its own context
   window. It is gated as CHECKPOINT (see [automation-levels.md](automation-levels.md)) so the user
   opts in each time.
8. Analyst teammates default to the `sonnet` model, not Opus, to keep a 4-teammate debate affordable.
9. The council reads the briefing and dossiers that already exist on disk. It does NOT re-gather from
   BrightData, except small targeted spot-checks an analyst needs to test a specific claim.
10. Keep the team to 4 teammates. Clean it up as soon as the addendum is written so no idle teammate
    keeps burning context.
