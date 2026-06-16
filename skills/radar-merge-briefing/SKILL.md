---
name: radar-merge-briefing
description: Merges all competitor dossiers and the change set into one briefing. Writes three local artifacts: the briefing markdown, a combined raw dossier JSON, and a dashboard data JSON. Use after radar-diff-changes and before radar-publish-notion and radar-render-dashboard. Every claim in the briefing carries a source URL.
---

# radar-merge-briefing

Automation level: AUTONOMOUS.

Goal: produce the single source of truth for this run on local disk, with a source URL on every
claim and no fabricated or stale-as-fresh values.

## Steps

1. Inputs from the caller: the run date, the run dossier directory, and the change set from
   `radar-diff-changes`. Read `radar.config.json` for business name, keywords, and freshness windows.

2. Load every dossier in the run directory.

3. Render the briefing markdown from the template at
   `${CLAUDE_PLUGIN_ROOT}/assets/briefing-template.md`. Fill every placeholder. For each competitor
   field, write the value, its source link, the fetched date, and the status. Where a field is
   `not_found` or `blocked`, say so plainly rather than leaving a gap or inventing a value. Build the
   "What changed" and "Focus list" sections from the ranked change set.

4. Enforce the rules as you write:
   - No claim without a source ([no-fabrication.md](../../rules/no-fabrication.md)).
   - Label freshness per section ([freshness.md](../../rules/freshness.md)).
   - No em or en dashes ([no-em-dashes.md](../../rules/no-em-dashes.md)); normalize any in quotes.

5. Write three files:
   - Briefing markdown: `outputs/briefings/<YYYY-MM-DD>-briefing.md`
   - Combined raw dossier JSON: `outputs/dossiers/<YYYY-MM-DD>/combined.json` (all dossiers plus the
     change set, the machine-readable record).
   - Dashboard data JSON: `outputs/dashboard/<YYYY-MM-DD>-data.json`, shaped for the dashboard
     template (see the shape below). `war_room` is null until `/radar-council` fills it.

6. Update `radar.config.json` `last_run` with the new paths and date, and roll the previous run's
   dossier dir so the next diff has a baseline.

### Dashboard data JSON shape

The dashboard at `${CLAUDE_PLUGIN_ROOT}/assets/dashboard-template.html` reads this exact shape. The
template computes missing KPIs itself, but populate `kpis` so the numbers match the briefing. SERP
rank values MUST be numbers (not strings) so the leaderboard renders. Sentiment values are one of
`positive`, `mixed`, `negative`, `unknown`. Each change carries a `type`
(`RISING`, `FALLING`, `PRICE`, `NEWS`, `NEW`, `STEADY`) and the `competitor` name.

```json
{
  "business_name": "Acme Co",
  "run_date": "2026-06-16",
  "keywords": ["primary keyword", "secondary keyword"],
  "kpis": { "competitors_tracked": 6, "rising_threats": 2, "price_moves": 1,
            "new_signals": 3, "cited_pct": 100 },
  "changes": [
    { "type": "RISING", "competitor": "Competitor One",
      "text": "moved to SERP #2 for \"primary keyword\" (was #5).", "source_url": "..." }
  ],
  "competitors": [
    {
      "name": "Competitor One", "domain": "competitor-one.com",
      "threat_level": "", "change": "Rising",
      "serp_rank": { "value": 2, "keyword": "primary keyword", "source_url": "...", "fetched_at": "...", "status": "ok" },
      "pricing": { "value": "Pro $49/user/mo", "source_url": "...", "fetched_at": "...", "status": "ok" },
      "positioning": { "value": "...", "source_url": "...", "fetched_at": "...", "status": "ok" },
      "news": { "value": "...", "source_url": "...", "fetched_at": "...", "status": "ok" },
      "sentiment": { "value": "mixed", "source_url": "...", "fetched_at": "...", "status": "ok" },
      "funding": { "value": null, "source_url": null, "fetched_at": "...", "status": "not_found" },
      "hiring": { "value": null, "source_url": null, "fetched_at": "...", "status": "not_found" }
    }
  ],
  "war_room": null
}
```

`threat_level` stays empty at gather time (low/medium/high is assigned by the council) and the
dashboard simply omits the threat chip and filter until it is set. `change` comes from
`radar-diff-changes` (New, Rising, Falling, Steady).

7. Return the three local paths to the caller.

## Edge cases

- A dossier file is malformed: skip it, note the competitor as "dossier unreadable this run," and
  continue. Do not crash the whole briefing.
- All competitors blocked: still produce a briefing that honestly says data could not be gathered and
  suggests checking the BrightData connection.
- Keep the dashboard data JSON in sync with the markdown so all three surfaces agree.
