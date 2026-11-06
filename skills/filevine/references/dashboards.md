# Filevine dashboards in Claude

Written for the Claude Dashboard artifact type as of 2026-11-06. When you create a dashboard, the type hands you its own instructions and reference files. They win over this file. This file covers only the Filevine part.

## 1. Decide first: live dashboard or one-off snapshot

| | Live dashboard | One-off snapshot |
|---|---|---|
| What it stores | Queries. Every open calls Filevine again. | Rows pulled once, as an attached file. |
| Use it when | It is a view someone will come back to: the default. | One meeting, one answer, a number that must stay "as of" a date, or a reader with no route to Filevine. |
| Freshness | Current on every open. | As of the pull. Date it in the title and `description`. |
| Who sees the data | Each reader, through their own Filevine connection (see section 5 for the owner-access option). | Everyone who can open the dashboard reads the file, whatever their own Filevine access. |
| Source form | `{kind: "call", connector: "Filevine", tool, args}` | `{kind: "file", url, name, format: "json"}`, with `fixed: true` and `updated: {at, by}` |

Default to live. Make a snapshot only when the person asks for a copy that stays as it is, or live cannot work. Say plainly that it is not live and why.

**Snapshot privacy.** An attached file is readable by everyone who can open the dashboard. Filevine rows can hold client data. Pull with `requestedFields`, aggregate to what the page shows, and leave out names unless the person asks for them. Say once that the file is readable by everyone with access to the dashboard.

**Snapshot recipe.** Run the Filevine reads in your session and page them. Aggregate to what the page shows where you can. Write a JSON array of row objects and upload it as the type describes. Then write the dataset (`title`, a `description` that says what and as of when, the `file` source, `updated`, `fixed: true`) and the page. The page follows the same marking rules as a live one.

## 2. Live dashboards: what Filevine needs

A live dashboard must only call `get_*` tools. Never put a write tool in a stored query. A stored query runs on every reader's open, so ask only for the rows and fields the page uses.

**Use `kind: "call"`.** The type's current form is `{kind: "call", connector, tool, args}`. The older `kind: "query"` form still runs, but the type tells new work to write `"call"`. The connector name is the one the person added it under: `Filevine`.

Projects (main source):

```json
{
  "kind": "call",
  "connector": "Filevine",
  "tool": "get_fv_app_Projects",
  "args": {
    "header_x_fv_orgid": 0,
    "excludeArchived": true,
    "limit": 1000,
    "offset": 0,
    "requestedFields": "projectId,phaseName,projectTypeId,createdDate,lastActivity"
  }
}
```

Use the real org ID from `post_fv_app_utils_GetUserOrgsWithToken` (a number, not a string). Project types, for names: `get_fv_app_ProjectTypes` with `{header_x_fv_orgid, limit: 100, requestedFields: "projectTypeId,name"}`. Both return `{content: {count, offset, limit, hasMore, items: [...]}}`.

**Every Filevine dataset needs a loader.** Filevine answers are not plain rows. The loader unwraps the answer, flattens ID objects, trims dates and follows pages. It only reshapes. Count, join and bucket in `dash.calc` so readers can see the formula. Keep IDs as strings and join on the string form. Filevine cannot aggregate, so `dash.calc` is the right place for counting and joining.

Projects loader:

```js
dash.loader('projects', {
  description: 'Filevine project list as one row per case, following pages',
  fn: async ({ answer, call, args }) => {
    const body = (a) => { const x = typeof a === 'string' ? JSON.parse(a) : a; return x && x.content ? x.content : x; };
    const id = (v) => (v && typeof v === 'object' ? v.native : v);
    const day = (s) => (s ? String(s).slice(0, 10) : null);
    const limit = args.limit || 1000;
    const rows = [];
    let page = body(answer);
    for (let n = 1; ; n++) {
      if (!page || !Array.isArray(page.items)) throw new Error('Filevine did not return a project list' + (page && page.message ? ': ' + page.message : ''));
      for (const p of page.items) rows.push({
        projectId: String(id(p.projectId)), phase: p.phaseName || 'No phase',
        projectTypeId: String(id(p.projectTypeId)), created: day(p.createdDate), lastActivity: day(p.lastActivity),
      });
      const cols = ['projectId', 'phase', 'projectTypeId', 'created', 'lastActivity'];
      if (!page.hasMore) return { columns: cols, rows, cut: false };
      if (n >= 10) return { columns: cols, rows, cut: true };
      page = body(await call('get_fv_app_Projects', { ...args, offset: (args.offset || 0) + n * limit }));
    }
  },
});
```

Project types loader (the join needs it):

```js
dash.loader('project_types', {
  description: 'Filevine project types as one row per type',
  fn: ({ answer }) => {
    const x = typeof answer === 'string' ? JSON.parse(answer) : answer;
    const page = x && x.content ? x.content : x;
    if (!page || !Array.isArray(page.items)) throw new Error('Filevine did not return project types' + (page && page.message ? ': ' + page.message : ''));
    const id = (v) => (v && typeof v === 'object' ? v.native : v);
    return { columns: ['projectTypeId', 'name'], rows: page.items.map((p) => ({ projectTypeId: String(id(p.projectTypeId)), name: p.name || 'Unnamed' })), cut: false };
  },
});
```

## 3. Building a live one

1. Create the dashboard once from the Dashboard type. Every later write goes to the URL it returns.
2. Read the type's own reference files before designing. They set the page rules and change over time, so do not copy page rules from this file. As of 2026-11-06 a new page opens with `<meta name="dash-one-writer">`, keeps fixed text in the HTML, and draws data-driven pieces as `data-draw` elements with `dash.draw`. `dash.onData` is for showing and hiding. Values go in with `textContent`, never `innerHTML`. No hard-coded numbers.
3. Run each query in your own session first to confirm arguments and response shape.
4. Write everything in one batch: metadata, one dataset per query, the page. Then read the type's `check` document and run each stored call once. Fix and say so if one fails.
5. Do not screenshot or browse to verify unless the user asks.
6. For chart styling, use a data-visualization skill. This file covers only the Filevine part.

## 4. Dashboards that worked

- Open cases by phase and by project type.
- New cases per month (last 12 months, including zero months).
- Idle cases: under 7, 7 to 30, 30 to 90, over 90 days since last activity.

Checked 2026-11-06 (the call returns the shape below; no dashboard built on it yet): open and overdue tasks per user. `get_fv_app_users_tasks` takes a `userId` and returns `noteId`, `targetDate`, `isCompleted`, `assigneeId` and `projectId`, plus `body` (free text that can hold client details) unless you pass `requestedFields`. Overdue means `targetDate` before today and `isCompleted` false. It is one call per user.

Untested ideas: upcoming deadlines and appointments (calls return empty lists on a project with none; the row shape is unseen). Read each tool's schema first.

Not possible through this connector: A/R aging and anything billing. Settlements, liens and medical specials exist only if the firm keeps them in its own fields (a vitals call or a section read gets them, one call per case). See SKILL.md section 3.

## 5. Sharing

Only the owner shares, from the dashboard's Share menu. Claude cannot change sharing. A new dashboard is private. Tell the person who can open it and what they will see.

**Live dashboard readers.** Each reader needs a Claude account. By default their page calls Filevine on their own connection, so each reader needs their own Filevine connection and access to those cases in Filevine. Otherwise a section shows an error.

**Owner-access option.** The type lets the owner decide whose access a connector uses: in Sources, the connector's menu, "Viewers use": their access or the owner's. With the owner's access, readers see the owner's results for the approved queries and need no Filevine connection of their own. The trade-off is that everyone who can open the dashboard sees what the owner's Filevine access shows for those queries. This is the owner's choice, made in Sources. Claude must not switch it on. Explain the trade-off first. Not yet tested with Filevine.

**Outsiders.** From the previous version of this file, not retested: connector dashboards cannot be shared by public link, and guests invited by email can open the page but connector data does not load for them. For outsiders, make a snapshot (section 1) and invite them to it, after checking what the file contains.

## 6. Gotchas

- The org ID is fixed in each stored query. A dashboard for another firm needs its own.
- `count` in a response is the number of items in that page, not the total. Count rows after paging.
- Do not follow `links.next`. Page with `offset += limit` while `hasMore` is true. (One live `next` link, on the users list, was malformed.)
- The loader's 10-page cap covers 10,000 projects. A firm near that size makes ten calls on every open of every reader's dashboard. Prefer a narrower query or a snapshot there. Not yet built or timed on a firm that large.
- A loader that stops at 10 pages returns `cut: true`. Say on the page when data was cut. Every reader's page repeats the calls on every open, so a large firm means many calls. Checked only on a firm of 33 projects.
- The staff list (`get_fv_app_Users`) needs `requestedFields` that include `user`, because the ID is nested there, and that also returns emails. Keep only the ID and name in the loader.
- A firm may contain test projects and test project types. Check before treating counts as real.
