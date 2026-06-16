# Rule: Subagents vs Agent Teams (use the right tool for each step)

This plugin deliberately uses BOTH of Claude Code's parallelism primitives, and never swaps them.
Picking the wrong one is a design error, not a style choice.

## The distinction

| | Subagents | Agent teams (experimental) |
| --- | --- | --- |
| Communication | Report results back to the caller ONLY. They never talk to each other. | Teammates message each other directly through a shared mailbox. |
| Coordination | The caller manages all work. | Shared task list; teammates self-claim and coordinate. |
| Context | Own context window; result returns to caller. | Own context window; fully independent session. |
| Best for | Focused gather/lookup where only the result matters. | Work that needs discussion, challenge, and convergence. |
| Token cost | Lower (result summarized back). | Higher (each teammate is a full Claude instance). |

## How this plugin applies it

### GATHER = subagents (`/radar`)

Gathering each competitor's dossier is embarrassingly parallel and needs zero cross-talk. Scout A
researching Competitor A has nothing to say to Scout B. So `/radar` spawns N `competitor-scout`
SUBAGENTS in one message; each one researches a single competitor, writes one cited JSON dossier, and
reports back. The orchestrator (not the scouts) merges them. Using an agent team here would add
mailbox and coordination overhead for no benefit and burn extra tokens.

### DEBATE = agent team (`/radar-council`)

Turning dossiers into strategy is the opposite problem. A pricing view, a product-gap view, a
threat view, and a devil's advocate must SEE and CHALLENGE each other to avoid anchoring on the first
plausible story. That requires teammates that message each other, which only an agent team provides.
So `/radar-council` creates a real experimental agent team of the 4 analyst roles, lets them debate
like a scientific dispute, and converges on recommended moves. Using plain subagents here would mean
four isolated opinions with no rebuttal, which is exactly the failure mode we are avoiding.

## The critical caveat (do not forget)

When a subagent definition is spawned as an agent-team TEAMMATE, its `skills` and `mcpServers`
frontmatter fields are NOT applied. A teammate loads MCP servers and skills from the USER's project
and user settings, the same as a regular session. This is why onboarding
([radar-onboard](../skills/radar-onboard/SKILL.md)) must verify the user's own BrightData and Notion
connection: the analyst teammates depend on the user's settings, not the plugin's `.mcp.json`. See
[agent-teams-setup.md](../references/agent-teams-setup.md) for the full setup and limitations.
