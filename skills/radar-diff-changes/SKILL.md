---
name: radar-diff-changes
description: Diffs this run's competitor dossiers against the previous run to find what materially changed (SERP moves, price changes, new news, sentiment shifts, new or dropped competitors). Produces a ranked change set the briefing and dashboard use. Use after all competitor-scout subagents have written their dossiers and before radar-merge-briefing.
---

# radar-diff-changes

Automation level: AUTONOMOUS.

Goal: compute an honest, freshness-aware change set between the current run and the last one.

## Steps

1. Read `radar.config.json` to find `last_run.dossier_dir` (the previous run) and the current run
   directory from the caller. If there is no previous run, this is the baseline: every competitor is
   marked `New` and there are no deltas to compute. Say so.

2. Load all dossier JSON files from both runs, matched by competitor domain (the stable key).

3. For each competitor and each field, compare ONLY when the field is within its recency window on
   BOTH runs (see [freshness.md](../../rules/freshness.md)). A comparison across a stale value is not
   a real change; skip it and note the field as "not comparable this run."

4. Classify each material change with a type the dashboard understands:
   - `RISING`: competitor improved SERP rank, gained funding, ramped hiring, sentiment up.
   - `FALLING`: competitor lost rank, negative news, sentiment down.
   - `PRICE`: pricing or packaging changed.
   - `NEWS`: a fresh in-window headline that did not exist last run.
   - `NEW`: a competitor appearing for the first time, or newly ranking for a tracked keyword.
   Each change carries the supporting `source_url` and `fetched_at` from the dossier. No source, no
   change entry.

5. Rank changes by materiality (rank jumps on core keywords and price moves rank highest), and
   produce a one-line change digest for the briefing headline plus the structured change array for
   the dashboard.

6. Return the change set to the caller. Do not write surfaces here; that is the merge and publish
   skills.

## Edge cases

- A competitor present last run but absent now: mark `dropped` in notes (do not invent a reason).
- A field blocked this run but present last run: report "could not verify this run" rather than a
  false `FALLING`.
- Timestamps missing on an old dossier: treat that field as not comparable.
