# Reference: Agent Teams Setup (for /radar-council)

`/radar-council` uses Claude Code's experimental agent-teams feature. This reference captures
everything the command and the user need, distilled from the official docs at
https://code.claude.com/docs/en/agent-teams.

## Prerequisites

1. **Version**: Claude Code v2.1.32 or later. Check with `claude --version`.
2. **Enable flag**: the feature is off by default. Turn it on by setting the environment variable
   `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` to `1`, in the shell or in settings.json:

   ```json
   {
     "env": {
       "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
     }
   }
   ```

`radar-setup` checks both prerequisites and tells the user exactly how to fix a missing one before
the council is ever attempted.

## Spawning teammates by name

A teammate can be spawned from any subagent definition by referencing its type name. This plugin
ships four analyst definitions in `agents/`:

- `pricing-analyst`
- `product-gap-analyst`
- `threat-sentiment-analyst`
- `devils-advocate`

When spawned as a teammate, the definition's `tools` allowlist and `model` are honored, and the
definition body is appended to the teammate's system prompt as extra instructions.

## The caveat that drives onboarding

When a subagent definition runs AS A TEAMMATE, its `skills` and `mcpServers` frontmatter fields are
NOT applied. Teammates load skills and MCP servers from the USER's project and user settings, the
same as a regular session. CLAUDE.md is read normally from the working directory.

Consequence: the analyst teammates can only spot-check BrightData and read Notion if the USER has
those MCP servers connected in their own settings. This is exactly why `radar-onboard` asks about and
live-tests the user's MCP connection, and records the working prefix. The plugin's `.mcp.json` alone
is not enough for teammates.

## Display modes

- `in-process` (default fallback): all teammates run in the main terminal. Shift+Down cycles through
  them; Ctrl+T toggles the task list.
- `tmux` / split panes: each teammate gets a pane. Requires tmux or iTerm2 with the it2 CLI.
- `auto` (default): split panes if already in tmux or iTerm2, otherwise in-process.

Set `teammateMode` in `~/.claude/settings.json`, or pass `--teammate-mode in-process` for one
session. The council works in any mode; in-process needs no extra setup.

## Lifecycle (how /radar-council uses it)

1. Confirm prerequisites and that the user wants to spend the tokens (CHECKPOINT).
2. Ask the lead to create ONE team and spawn the 4 analyst teammates by name, each with a spawn prompt
   pointing at the latest briefing and dossiers on disk and its specific lens.
3. Let them debate: each teammate challenges the others' conclusions through direct messages, like a
   scientific dispute, and they converge on recommended moves.
4. The lead synthesizes the war-room addendum.
5. Publish via `radar-publish-council`.
6. Clean up the team (always via the lead, never a teammate). Cleanup fails if a teammate is still
   running, so shut teammates down first.

## Known limitations to plan around

- No session resumption with in-process teammates: `/resume` and `/rewind` do not restore them. If a
  resumed lead tries to message gone teammates, spawn new ones.
- Task status can lag; a stuck task may need a manual status nudge.
- Shutdown can be slow (teammates finish the current tool call first).
- One team at a time per lead; no nested teams; the lead is fixed for the team's lifetime.
- Permissions are set at spawn from the lead's mode.
- Split panes are unsupported in VS Code's integrated terminal, Windows Terminal, and Ghostty; those
  fall back to in-process.

See [subagents-vs-agent-teams.md](../rules/subagents-vs-agent-teams.md) for why the debate uses a team
and the gather uses subagents.
