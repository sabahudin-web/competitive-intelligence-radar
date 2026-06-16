# Rule: Freshness (recency windows + fetched_at)

Old data presented as current is a quiet form of fabrication. This rule keeps the radar honest about
time.

## The rule

1. Every gathered field carries a `fetched_at` ISO 8601 UTC timestamp recording when the source was
   actually retrieved this run.
2. Every field is judged against a recency window for its data type. A source older than its window
   is marked `status: "stale"` and is NOT presented as a current fact in the briefing headline; it
   may appear in a clearly labeled "as of <date>" footnote.

## Recency windows (defaults, overridable in radar.config.json)

| Data type                         | Window        | Notes                                              |
| --------------------------------- | ------------- | -------------------------------------------------- |
| Pricing / plans                   | this run only | Always re-fetched live. Never carried over.        |
| SERP rank for my keywords         | this run only | Always re-fetched live.                            |
| Homepage / product positioning    | this run only | Always re-fetched live.                            |
| News                              | <= 30 days    | Older items are context, not "news."               |
| Funding / financials              | <= 90 days    | Rounds and filings move slowly.                    |
| Sentiment (social, reviews)       | <= 30 days    | Mood shifts fast; old posts mislead.               |
| Hiring / job listings             | <= 30 days    | Signals current direction.                         |

## In practice

- The diff step ([radar-diff-changes](../skills/radar-diff-changes/SKILL.md)) compares only fields
  whose `fetched_at` is within window on BOTH runs, so a "change" is a real change and not a stale
  comparison.
- The dashboard and briefing display each section's freshness ("fetched 2026-06-16, news window 30d").
- If every source for a competitor is stale or blocked, the scout still returns a dossier with
  honest `status` values rather than inventing freshness.

See [no-fabrication.md](no-fabrication.md) for the citation requirement these timestamps support.
