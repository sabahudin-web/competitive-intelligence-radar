# Reference: Notion Schema (proposed)

This is the schema `radar-provision-notion` SHOWS the user for approval before creating anything.
Nothing is created in Notion until the user says yes. See
[automation-levels.md](../rules/automation-levels.md).

The Notion MCP `create-database` tool takes SQL DDL (a `CREATE TABLE` statement). Column names are
double quoted; type options use single quotes. Two databases are proposed, both created under a
parent page the user chooses during onboarding.

## Database 1: Competitors

One row per competitor, upserted every run (matched by Domain). This is where week-over-week change
tracking lives.

| Property | Type | Purpose |
| --- | --- | --- |
| Name | TITLE | Competitor name |
| Domain | URL | Canonical domain, used as the upsert key |
| SERP Rank | NUMBER | Best position for my tracked keywords this run |
| Pricing Summary | RICH_TEXT | Short plan/price summary |
| Pricing Source | URL | Cited source for the pricing summary |
| Sentiment | SELECT | positive / mixed / negative / unknown |
| Latest News | RICH_TEXT | Most recent in-window headline |
| News Source | URL | Cited source for the news |
| Funding / Financials | RICH_TEXT | Latest round, valuation, or filing note |
| Hiring Signal | RICH_TEXT | Notable open roles or hiring direction |
| Threat Level | SELECT | low / medium / high |
| Change | SELECT | New / Rising / Falling / Steady (vs last run) |
| Last Fetched | DATE | Freshness stamp for the row |

DDL:

```sql
CREATE TABLE (
  "Name" TITLE,
  "Domain" URL,
  "SERP Rank" NUMBER,
  "Pricing Summary" RICH_TEXT,
  "Pricing Source" URL,
  "Sentiment" SELECT('positive':green, 'mixed':yellow, 'negative':red, 'unknown':gray),
  "Latest News" RICH_TEXT,
  "News Source" URL,
  "Funding / Financials" RICH_TEXT,
  "Hiring Signal" RICH_TEXT,
  "Threat Level" SELECT('low':green, 'medium':yellow, 'high':red),
  "Change" SELECT('New':blue, 'Rising':red, 'Falling':green, 'Steady':gray),
  "Last Fetched" DATE
)
```

## Database 2: Briefings

One row (and one page) per weekly run. The page body holds the full briefing markdown, and the
war-room addendum is appended to the same page by `radar-publish-council`.

| Property | Type | Purpose |
| --- | --- | --- |
| Title | TITLE | "Radar Briefing YYYY-MM-DD" |
| Run Date | DATE | When the run happened |
| Competitors Tracked | NUMBER | Count in this run |
| Key Changes | RICH_TEXT | One-line digest of the biggest deltas |
| War Room | CHECKBOX | Checked once a council addendum is appended |

DDL:

```sql
CREATE TABLE (
  "Title" TITLE,
  "Run Date" DATE,
  "Competitors Tracked" NUMBER,
  "Key Changes" RICH_TEXT,
  "War Room" CHECKBOX
)
```

## Notes for the implementer

- `create-database` returns a `<data-source>` ID. Store both database IDs and both data source IDs in
  `radar.config.json` (`notion.competitors_db`, `notion.competitors_ds`, `notion.briefings_db`,
  `notion.briefings_ds`). Pages are created under the data source ID, not the database ID.
- Upsert pattern for Competitors: `notion-search` or `query_data_sources` by Domain; if a page
  exists, `notion-update-page`; else `notion-create-pages`. This keeps provisioning and publishing
  idempotent.
- All text written here obeys [no-em-dashes.md](../rules/no-em-dashes.md) and
  [no-fabrication.md](../rules/no-fabrication.md): every value that came from the web keeps a source
  URL in its paired Source column or inline in the page body.
