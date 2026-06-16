---
name: radar-plan-targets
description: Builds the per-competitor scout job list for a radar gather run. Reads radar.config.json, enforces the cap of 8 competitors and waves of 8, and emits one ready-to-spawn job spec per competitor (competitor, keywords, geo, scout depth, brightdata prefix, output path). Use as the first step of /radar, before spawning competitor-scout subagents.
---

# radar-plan-targets

Automation level: AUTONOMOUS. This is read-and-plan only; it spawns nothing.

Goal: turn the config into a clean list of scout jobs the `/radar` command will fan out in one
message. See [cost-control.md](../../rules/cost-control.md) for the caps.

## Steps

1. Read `radar.config.json`. If missing or missing a verified `brightdata_prefix`, stop and route the
   user to `/radar-setup`.

2. Establish the run id and output directory: a date-stamped run, for example
   `outputs/dossiers/<YYYY-MM-DD>/`. Create the directory. Each scout writes one file
   `<competitor-slug>.json` there.

3. Apply caps: take at most the first 8 competitors. If more were configured, note which were
   skipped. Plan waves of at most 8 (normally one wave given the cap).

4. For each competitor, build a job spec the scout understands:
   - `COMPETITOR`: name + domain
   - `MY_KEYWORDS`: from config
   - `GEO`: config geo
   - `SCOUT_DEPTH`: the three flags from config
   - `BRIGHTDATA_PREFIX`: the verified prefix from config
   - `OUTPUT_PATH`: `outputs/dossiers/<run-date>/<slug>.json`
   - `FRESHNESS`: the news/funding/sentiment day windows (config overrides or the defaults in
     [freshness.md](../../rules/freshness.md))

5. Return the job list and the run directory to the caller. Do not gather anything yourself.

## Edge cases

- Zero competitors configured: stop and tell the user to add competitors via `radar-onboard`.
- Duplicate domains: dedupe, keep the first.
- A competitor missing a domain: keep the name, let the scout resolve the domain via search; mark it
  so the scout knows to confirm the domain first.
