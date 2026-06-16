---
name: radar-publish-notion
description: Publishes the run to Notion. Upserts one row per competitor in the Competitors database (matched by domain, with a Change field for week-over-week tracking) and creates a Briefings page holding the full briefing. Use after radar-merge-briefing. Requires the databases to exist (radar-provision-notion). Writing rows to an approved database is autonomous.
---

# radar-publish-notion

Automation level: AUTONOMOUS once the databases exist and were approved at setup. If they do not
exist, route to `radar-provision-notion` (CHECKPOINT). See
[automation-levels.md](../../rules/automation-levels.md).

Goal: reflect the run in Notion with honest change tracking, idempotently.

## Steps

1. Read `radar.config.json`. Need `notion_prefix`, `competitors_ds`, `briefings_ds`. If any is
   missing, stop and route to `radar-provision-notion`.

2. Load the combined dossier JSON and the change set produced by `radar-merge-briefing`.

3. Upsert each competitor into the Competitors data source (matched by Domain):
   - Find the existing row: `<notion_prefix>notion-search` or query the data source by Domain.
   - If found, `<notion_prefix>notion-update-page` with this run's values and set `Change` from the
     change set (New, Rising, Falling, Steady) and `Last Fetched`.
   - If not found, `<notion_prefix>notion-create-pages` under `competitors_ds`.
   - Put each cited value in its column and the matching source URL in its paired Source column. Set
     Threat Level from the threat reading if present, else leave it unset rather than guessing.

4. Create the Briefings page: `<notion_prefix>notion-create-pages` under `briefings_ds` with Title
   "Radar Briefing <date>", Run Date, Competitors Tracked, Key Changes (the one-line digest), War
   Room unchecked. The page body is the briefing markdown from `radar-merge-briefing` (Notion
   markdown). Store the returned page id in `last_run.briefing_notion_page_id` so
   `radar-publish-council` can append to it.

5. Obey [no-em-dashes.md](../../rules/no-em-dashes.md) and
   [no-fabrication.md](../../rules/no-fabrication.md) in everything written: every web value keeps its
   source.

6. Return the Notion page link to the caller.

## Edge cases

- A competitor matches more than one existing row (dirty data): update the most recently edited and
  note the duplicate; do not create a third.
- Notion rate limit or transient error: retry the single failed write a couple of times, then report
  exactly which competitor failed so the user can re-run; the upsert keying makes a re-run safe.
- Page body too long for one create: create the page with the summary, then append the rest with an
  update.
