# Rule: Automation Levels

Each skill and command declares an automation level so the user always knows when the radar will act
on its own and when it will stop and ask.

## Levels

- **AUTONOMOUS**: run end to end without pausing. Use for read, gather, diff, merge, render, and
  publish-to-surfaces-the-user-already-approved steps.
- **CHECKPOINT**: stop, show the user what is about to happen, and wait for an explicit yes before
  proceeding. Use for actions that create external state, cost real money at scale, or are hard to
  undo.

## What is AUTONOMOUS

- `/radar` (the weekly gather) and all of its sub-skills: `radar-plan-targets`,
  `radar-diff-changes`, `radar-merge-briefing`, `radar-render-dashboard`.
- `radar-publish-notion` once the databases already exist and were approved at setup. Writing rows to
  an approved database is routine.

## What is CHECKPOINT (must ask first)

1. **Notion database creation** (`radar-provision-notion`): before creating ANY database, show the
   proposed schema and ask "do you like this, or want changes?" Build only after approval. This is a
   hard requirement, not a preference.
2. **`/radar-council`**: the agent-team debate is token heavy. Always confirm the user wants to spend
   the tokens before spawning the team. See [cost-control.md](cost-control.md).
3. **MCP connection during onboarding** (`radar-onboard`): ask the user whether BrightData and Notion
   are connected, and stop with instructions if a live test fails. Do not assume.
4. **Publishing to GitHub** (packaging): never create or push a public repo without explicit
   confirmation.

## How to checkpoint

State plainly what you are about to do, what it will cost or create, and the single decision you need.
Then wait. Do not bundle the question with other output that buries it.
