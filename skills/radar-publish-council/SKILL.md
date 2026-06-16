---
name: radar-publish-council
description: Publishes the Strategic War Room addendum from /radar-council to all three surfaces. Appends the addendum to the local briefing markdown, fills the war_room section of the dashboard data and re-renders the dashboard, and appends the addendum to the Notion briefing page while checking its War Room box. Use after the agent team has converged on recommended moves.
---

# radar-publish-council

Automation level: AUTONOMOUS (the CHECKPOINT was the decision to run the council at all).

Goal: get the war-room addendum onto the same three surfaces the gather run published to, so the
strategy lives next to the evidence.

## Inputs

The synthesized addendum from the council lead, structured as:

```json
{
  "summary": "One paragraph of the consensus, no em dashes.",
  "moves": [
    { "title": "Concrete move", "rationale": "Why, with the supporting source or dossier field." }
  ],
  "dissents": [ "Any move a teammate still disputes, and why." ]
}
```

## Steps

1. Read `radar.config.json` `last_run` for the briefing markdown path, the dashboard data path, and
   `briefing_notion_page_id`.

2. Local markdown: append a "Strategic War Room" section to
   `outputs/briefings/<date>-briefing.md` below the addendum marker. Render the summary, each move as
   a titled bullet with its rationale and source, and a "Dissents and risks" list. Keep every claim
   cited and free of em or en dashes.

3. Dashboard: set `war_room` in the dashboard data JSON (`summary` and `moves[]`). If the council
   assigned threat levels (low / medium / high) from the threat ranking, also write each into the
   matching `competitors[].threat_level` so the dashboard threat chips and the threat filter light
   up. Then re-run the inject step from `radar-render-dashboard` so
   `outputs/dashboard/<date>-dashboard.html` and `index.html` now show the war room.

4. Notion: append the same addendum to the Briefings page using
   `<notion_prefix>notion-update-page` on `briefing_notion_page_id`, and set the page's `War Room`
   checkbox to true.

5. Confirm all three surfaces updated and return their paths and the Notion link.

## Edge cases

- The council ran but no briefing exists yet for this date: stop and tell the user to run `/radar`
  first; the council analyzes an existing briefing.
- Notion page id missing (gather did not publish to Notion): update the local markdown and dashboard,
  and tell the user Notion was skipped because the briefing was never published there.
- Re-running after an earlier council on the same briefing: replace the existing war-room section
  rather than appending a second one, so the surfaces do not accumulate duplicates.
