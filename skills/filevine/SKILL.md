---
name: filevine
description: Use when connecting Claude to Filevine, when Filevine sign-in fails, when asked what Claude can or cannot pull from Filevine (billing, time, trust, settlements), when reading Filevine projects, tasks, deadlines or notes, or when building a dashboard from Filevine data.
license: Apache-2.0
---

# Filevine with Claude

Filevine has its own official MCP connector. The connector lists its own tools, so check the live tool list before saying what is possible. If this file disagrees with the live tools, the live tools win. Last checked 2026-11-06.

## Connect

Filevine's guide: https://support.filevine.com/hc/en-us/articles/56321309703707-Filevine-Claude-Connector

- Add Filevine from Customize > Connectors in claude.ai (desktop or web), sign in on the Filevine page, pick the tenant, Approve. Each person connects for themselves and gets their own Filevine permissions.
- Do not add it by URL. `claude mcp add` fails with "does not support dynamic client registration" because Filevine only allows pre-registered apps. Connect in claude.ai and it appears in Claude Code too.
- The connection ends with the Filevine session (one day by default; the firm's admin can extend it to three months). Reconnect first. If sign-in still fails, ask the firm's admin to check the user's access.

## Org ID

Almost every tool needs `header_x_fv_orgid`. Call `post_fv_app_utils_GetUserOrgsWithToken` (no parameters) and read `content.orgs[]`. If there is more than one org, ask which.

## What it cannot do

- Timekeeping, billing, payments, the Client Portal and some document functions are not covered. Do not plan a query for unbilled time, invoices or trust balances. Point to Report Builder exports, or DataBridge if the firm has it. Never estimate.
- Settlements, liens and medical specials exist only if the firm keeps them in its own fields. Reading them costs one call per case. Say so before starting.
- It does cover reading and writing most other records, finding, moving, renaming and locking documents, and reading deadlines.

## Habits

- It can write and delete. Read first, and confirm with the user before any `post_`, `patch_`, `put_` or `delete_` call.
- Pass `requestedFields`. It keeps answers small and leaves out client names and personal data.
- IDs are objects like `{"native": 12345}`; read `.native`. `count` is the page size, not the total. Page with `offset += limit` while `hasMore` is true.
- `excludeArchived: true` on the project list means open cases. The list already carries phase, type, created date and last activity, so do not fetch those per case.
- A firm may contain test cases. Check before trusting counts.

More on response shapes: `references/data-shapes.md`.

## Dashboards

Read `references/dashboards.md`.

## Firm model

Every firm sets up project types, phases and fields differently. To keep what you learn about one firm between chats, read `references/firm-model.md`.
