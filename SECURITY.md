# Security policy

## Reporting a vulnerability

If you find a security issue in Competitive Intelligence Radar, please report it privately. Do not open a
public issue for it.

Contact Sabahudin Murtic on LinkedIn: https://www.linkedin.com/in/sabahudin-murtic/

Please include:

- What the issue is and where it lives (file, command, skill, or MCP call).
- Steps to reproduce it.
- What an attacker could do with it.

## What to expect

- A reply acknowledging your report as soon as it is seen.
- An honest assessment of whether it is in scope and how it will be handled.
- Credit for the find if you want it, once a fix is out.

## Handling secrets

This plugin never commits your secrets. Your BrightData token lives in your environment as
`BRIGHTDATA_API_TOKEN`, and your business context lives in `radar.config.json`, which is gitignored. If you
ever see a real token, a personal email, or business data staged for commit, stop and remove it before you
push. The `.gitignore` is set up to keep `.env`, `radar.config.json`, and everything under `outputs/` out of
the repo.

## Supported versions

This is an actively maintained plugin. Security fixes land on `main`. Older tags are not patched unless
stated otherwise.
