---
name: filevine
description: Use when connecting Claude to Filevine, when Filevine sign-in or API access fails, when asked what Claude can or cannot pull from Filevine (billing, time, trust, settlements), or when reading Filevine projects, tasks, deadlines or notes. For live dashboards, also read references/dashboards.md.
license: Apache-2.0
---

# Filevine with Claude

Filevine has its own official MCP connector. Do not send people down the client-ID-and-secret API route. Do not list the connector's tools from memory: it lists its own, so check the live tool list before saying what is possible. If this file disagrees with the live tools, the live tools win. Last checked 2026-11-05.

## 1. Get connected

Filevine's own guide: https://support.filevine.com/hc/en-us/articles/56321309703707-Filevine-Claude-Connector (comparison with LOIS: https://support.filevine.com/hc/en-us/articles/56475776327707-LOIS-and-Filevine-s-Claude-Connector-Capabilities).

- Each person connects for themselves. The connector acts as that user, with that user's Filevine permissions.
- In Claude desktop or Claude on the web: Customize > Connectors > add Filevine > Connect. Click Continue connecting, sign in at the Filevine Identity (FVID) page, pick the tenant if asked, review the scopes, and Approve.
- Add it from that Connectors list, not by URL. In Claude Code, `claude mcp add --transport http` with the server URL fails with "does not support dynamic client registration", because Filevine's sign-in server (identity.filevine.com) only allows pre-registered apps such as Claude's own connector. Connect Filevine in claude.ai; it then appears in Claude Code under the same account.
- By default every tool is set to "needs approval". Tool permissions can be changed from the connector's entry in Claude.
- The connection lasts as long as the Filevine session. Default is one day. The firm's admin can extend it up to three months. If it stops working, reconnect first.
- If sign-in still fails, ask the firm's Filevine admin to check the user's access. Do not send people to client IDs and secrets; that is a different, manual API route.
- Tool names look like `get_fv_app_Projects` and `post_fv_app_Notes`.

## 2. Find the org ID first

Almost every tool needs `header_x_fv_orgid`.

1. Call `post_fv_app_utils_GetUserOrgsWithToken` (no parameters).
2. Read `content.orgs[]` (each has `orgId`, `name`). A person can belong to several orgs; pick the right one, or ask.

## 3. What the connector does not cover

- **Not covered, per Filevine:** timekeeping, billing, payments, the Client Portal, and certain document functions. That means no invoices, time entries, unbilled time, trust accounting or rate schedules. Do not plan a query for them.
- **Covered:** reading and writing the other records, finding a document by filename, folder or metadata, moving, copying, renaming and locking documents, and reading deadlines already on a project.
- Money figures (settlements, liens, medical specials) exist only if the firm keeps them in custom form or collection fields on each case. Reading those takes one call per project: slow and costly. Say so before starting.
- When data is missing, say it is missing and point to Filevine's own routes (Report Builder exports, or DataBridge if the firm has it). Never estimate.
- This connector works inside Claude only. Other BI tools cannot use it.

## 4. Habits that keep it safe and fast

- The connector can write and delete, not only read. Read first. Confirm with the user before any `post_`, `patch_`, `put_` or `delete_` call.
- Pass `requestedFields` on list calls. It keeps answers small and leaves out client names and other personal data. Add names only when needed.
- The project list already includes phase, type, created date and last activity. Do not fetch those per case.
- IDs come back as objects like `{"native": 12345}`. Read `.native`.
- Dates are ISO UTC strings. Take the first 10 characters for the day.
- List calls return rows under `content.items`, with `hasMore`. Page with `offset += limit` while `hasMore` is true (`limit` 1000 worked for projects).
- `excludeArchived: true` means open cases.
- A firm may contain test cases. Check before treating counts as real.
- Tools that run per project are expensive across a whole firm. Use them sparingly.
- Load a tool's schema and run one small read-only call before relying on its arguments. Do not guess argument names.
- Results over about 50,000 characters arrive as a file, not in context. `requestedFields` avoids that.

## 5. Dashboards

For any dashboard from Filevine data, read `references/dashboards.md`. It covers when to build a live dashboard (queries that run on every open) and when a one-off snapshot (rows stored as a file) fits better, how to build each, and how sharing works.

## What has been tested

Tested 2026-11-05 and 2026-11-06 against a 33-project firm: `get_fv_app_Projects` (including `limit` 1000, paging, `excludeArchived`), `get_fv_app_ProjectTypes`, `post_fv_app_utils_GetUserOrgsWithToken`, `get_fv_app_users_tasks`, `get_fv_app_ProjectTypes_sections`, `get_fv_app_Users`, and the dashboard loader code. Deadlines and appointments were called on one project with none, so their row shape is unseen. Contacts, notes, documents, forms and collections are untested. Not tested: firms with more than 1,000 projects, shared-connector dashboards, and guest access. Read a tool's schema before use.
