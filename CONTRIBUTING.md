# Contributing to Competitive Intelligence Radar

Thanks for taking the time. This is a focused Claude Code plugin, so contributing is simple.

## Before you start

Read [CLAUDE.md](CLAUDE.md). It explains the architecture and the hard rules that every change must
respect. The rules live in [rules/](rules/) and they are not optional:

- Every factual claim in a generated artifact carries a real source URL and a fetch time. No fabrication.
- Never use an em dash or an en dash anywhere, including code and docs. Use commas, colons, or hyphens.
- The gather uses subagents. The council uses an agent team. Do not swap one for the other.

## How to propose a change

1. Fork the repo and create a branch off `main`: `git checkout -b your-change`.
2. Make the change. Keep it focused: one idea per pull request.
3. Test it (see below).
4. Open a pull request. Describe the problem it solves and how you tested it.

## Testing your change

- Validate the three JSON config files parse: `node -e "JSON.parse(require('fs').readFileSync('.claude-plugin/plugin.json'))"` (repeat for `.claude-plugin/marketplace.json` and `.mcp.json`).
- Grep the repo for an em dash (U+2014) and an en dash (U+2013). There should be zero hits.
- For a flow change, dry-run the affected command (`/radar`, `/radar-council`, or `/radar-setup`) and confirm all three output surfaces still publish.

## Reporting a bug or asking for a feature

Open an issue. Say what you expected, what happened, and the steps to reproduce it. A real example beats a
description every time.

## Contact

Questions? Reach Sabahudin Murtic on LinkedIn: https://www.linkedin.com/in/sabahudin-murtic/
