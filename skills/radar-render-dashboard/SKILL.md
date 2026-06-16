---
name: radar-render-dashboard
description: Renders the self-contained local HTML dashboard for a run by injecting the run's data JSON into the dashboard template. Produces one HTML file with inline CSS and JS and no external requests that the user can open directly in a browser. Use after radar-merge-briefing, as the last surface in a /radar run.
---

# radar-render-dashboard

Automation level: AUTONOMOUS.

Goal: produce one double-clickable HTML file that shows the run, works offline, and needs no build
step or dependencies.

## Steps

1. Inputs: the run date and the dashboard data JSON path from `radar-merge-briefing`
   (`outputs/dashboard/<YYYY-MM-DD>-data.json`). Read it.

2. Read the template at `${CLAUDE_PLUGIN_ROOT}/assets/dashboard-template.html`.

3. Inject the data: replace the single placeholder `{{RADAR_DATA_JSON}}` (inside the
   `<script id="radar-data" type="application/json">` block) with the exact JSON contents. Do not add
   script tags, CDN links, or external fonts. The data must be inline JSON so the file is fully
   self-contained.

4. Write the result to `outputs/dashboard/<YYYY-MM-DD>-dashboard.html`. Also write or overwrite
   `outputs/dashboard/index.html` as a copy of the latest run for a stable "open this" path.

5. Confirm the absolute path and tell the user they can open it in any browser. Offer to open it
   (on macOS, `open <path>`), but do not require it.

## Edge cases

- The data JSON contains characters that could break the inline script: the template reads the JSON
  with JSON.parse from a typed script block, so standard JSON is safe. Ensure the JSON does not
  contain the literal sequence that closes a script tag; if a value does, escape the slash.
- War room not yet present: leave `war_room` null; the dashboard shows a prompt to run
  `/radar-council`. `radar-publish-council` re-renders later with the addendum filled in.
- No write permission to outputs: report the error and the intended path rather than failing
  silently.
