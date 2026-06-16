# Rule: No Fabrication (cite every claim)

This is the most important rule in the plugin. The product promise is "correct, up to date web
data." A single invented number destroys that promise.

## The rule

Every factual claim about a competitor MUST carry a real `source_url` that was actually fetched
during this run, plus the `fetched_at` timestamp of that fetch. No exceptions.

A "factual claim" is any value that describes the outside world: a price, a plan name, a SERP
position, a headcount, a funding round, a review score, a quote, a job opening, a product feature.

## What this means in practice

- If a BrightData call did not return a value, the field is `null` with `status: "not_found"`.
  Never guess, never round, never "estimate from memory."
- Never reuse a URL you did not fetch this run just to satisfy the schema. A stale or unrelated URL
  is a fabrication.
- Do not infer one competitor's pricing from another's. Each value stands on its own source.
- Quotes (reviews, posts, news) are copied verbatim and attributed to the exact URL they came from.
- If a page was blocked and even the scraping browser fallback failed, record
  `status: "blocked"` with the attempted URL. That is honest. A made up value is not.
- Model knowledge is NOT a source. Your training data is never a `source_url`.

## Dossier field shape (enforced everywhere)

```json
{
  "value": "Pro plan: $49/user/month",
  "source_url": "https://competitor.com/pricing",
  "fetched_at": "2026-06-16T14:03:00Z",
  "status": "ok"
}
```

`status` is one of: `ok`, `not_found`, `blocked`, `stale`.

## Verification

The packaging step greps generated briefings and dossiers. Any claim line without an accompanying
source URL is a build failure. See [no-em-dashes.md](no-em-dashes.md) for the companion text rule.
Freshness of each source is governed by [freshness.md](freshness.md).
