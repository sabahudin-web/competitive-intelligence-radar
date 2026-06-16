---
description: The on-demand strategic war room. Spins up a REAL experimental agent team of 4 analyst teammates (pricing, product gap, threat and sentiment, devil's advocate) over the latest briefing. They message and challenge each other like a scientific debate, converge on recommended moves, and the addendum is published to all three surfaces. Token heavy; asks before it runs.
---

# /radar-council

You are the team LEAD. This command is CHECKPOINT: it is token heavy because each teammate is a
separate Claude instance. You use a real AGENT TEAM here, never plain subagents, because the analysts
must see and challenge each other. See [subagents-vs-agent-teams.md](../rules/subagents-vs-agent-teams.md)
and [agent-teams-setup.md](../references/agent-teams-setup.md).

## Preconditions

1. A briefing exists for the latest run (`radar.config.json` `last_run.briefing_path`). If not, stop
   and tell the user to run `/radar` first.
2. Agent teams are enabled: Claude Code v2.1.32+ and `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. If not,
   show the fix from the setup reference and stop.
3. The user's own BrightData and Notion MCP are connected (teammates load MCP from USER settings, not
   plugin frontmatter). The verified prefixes are in the config; if they are stale, suggest re-running
   `/radar-setup`.

## CHECKPOINT

Tell the user plainly: the council spawns 4 Sonnet teammates that debate, which uses significantly
more tokens than a normal run. Ask for an explicit yes before spawning. Do not proceed on silence.

## Sequence

1. Create ONE agent team and spawn 4 teammates BY NAME from this plugin's agent definitions:
   `pricing-analyst`, `product-gap-analyst`, `threat-sentiment-analyst`, `devils-advocate`. Give each
   a spawn prompt that includes: the absolute path to the latest briefing markdown and the combined
   dossier JSON, the live BrightData prefix for light spot-checks, its specific lens, and the
   instruction to message and challenge the other teammates.

2. Run the debate. Have the analysts post opening reads, then actively challenge each other (the
   devil's advocate tries to disprove the others; each defends with cited evidence or concedes).
   Steer toward convergence; tell teammates to wait for each other rather than racing ahead. Keep it
   to a focused debate, not an endless one.

3. Synthesize the Strategic War Room addendum yourself as lead: a short consensus summary, the top
   recommended moves (each with a one-line rationale and the supporting source or dossier field), and
   a dissents-and-risks list for anything still disputed. No fabrication, no em or en dashes.

4. Run the `radar-publish-council` skill with that structured addendum to append it to the local
   briefing markdown, fill the dashboard war room and re-render, and append it to the Notion briefing
   page (checking its War Room box).

5. Clean up the team: shut down the teammates, then clean up the team as the lead (never from a
   teammate). Confirm cleanup succeeded.

6. Report the three updated surfaces and a two-line summary of the recommended moves.

## Edge cases

- A teammate stalls or errors: check it, give direct instructions, or spawn a replacement. Do not let
  the team run unattended too long.
- Cleanup fails because a teammate is still running: shut the teammate down first, then retry cleanup.
- Council requested before any gather: stop and route to `/radar`.
