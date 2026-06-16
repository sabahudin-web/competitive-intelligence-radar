---
name: radar-provision-notion
description: Creates the two Notion databases (Competitors and Briefings) the radar publishes to, but ONLY after SHOWING the proposed schema and getting the user's explicit approval. Use during /radar-setup after onboarding, or when radar-publish-notion reports the databases are missing. Idempotent: if the databases already exist in the config, it verifies them instead of creating duplicates. Never creates anything in Notion before the user says yes.
---

# radar-provision-notion

Automation level: CHECKPOINT. Showing the schema and getting approval BEFORE creating any database
is a hard requirement. See [automation-levels.md](../../rules/automation-levels.md).

Prerequisite: `radar.config.json` exists with a verified `notion_prefix`. If not, route the user to
`radar-onboard`.

## Steps

1. Idempotency check first. Read `radar.config.json`. If `notion.competitors_ds` and
   `notion.briefings_ds` are already set, verify they still resolve with a `notion-fetch`. If both
   resolve, report "Notion already provisioned" and stop. Do not create duplicates.

2. SHOW the proposed schema for both databases, exactly as laid out in
   [notion-schema.md](../../references/notion-schema.md): the Competitors database columns and the
   Briefings database columns, in a readable table. Then ask, in plain words:
   "Do you like this schema, or do you want changes (rename, add, or drop any field)?"
   STOP and wait for an answer. Do not proceed on silence.

3. Incorporate any requested changes into the DDL. Re-show if the changes are non-trivial and confirm
   again.

4. Only after approval, create the databases under the chosen parent
   (`notion.parent_page_id`, or workspace level if empty), using the Notion `create-database` tool
   (`<notion_prefix>notion-create-database`) with the approved `CREATE TABLE` DDL. Create the
   Competitors database first, then the Briefings database.

5. Capture the returned database id AND the `<data-source>` id for each, and write all four into
   `radar.config.json` (`competitors_db`, `competitors_ds`, `briefings_db`, `briefings_ds`). Pages
   are later created under the data source ids.

6. Confirm to the user with the Notion links and a one-line note that publishing is now enabled.

## Edge cases

- Parent page not accessible to the Notion integration: stop and tell the user to share the parent
  page with the connected Notion integration, then re-run.
- User wants to reuse an existing database: accept its database url, `notion-fetch` it to get the
  data source id, store the ids, and skip creation. Verify the columns cover what publishing needs;
  warn about any missing column rather than silently failing later.
- Partial creation (Competitors created, Briefings failed): save what succeeded so a re-run is
  idempotent and only creates the missing one.
