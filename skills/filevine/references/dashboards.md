# Live Filevine dashboards in Claude

Written for the Claude Dashboard artifact type as of 2026-11-05. Check the feature still exists before starting.

A dashboard stores queries, not numbers. Each time someone opens it, it calls the connector live with that viewer's own sign-in. Clicking a number shows the query, the calculation and the raw rows.

**A dashboard must only call `get_*` tools.** Never put a write tool in a stored query.

## Data sources

`get_fv_app_Projects` (main source):

```json
{
  "header_x_fv_orgid": <orgId>,
  "excludeArchived": true,
  "limit": 1000,
  "offset": 0,
  "requestedFields": "projectId,phaseName,projectTypeId,createdDate,lastActivity"
}
```

`get_fv_app_ProjectTypes` (names for type IDs): `{header_x_fv_orgid, limit: 100, requestedFields: "projectTypeId,name"}`. Join on `projectTypeId.native`.

Both return `{content: {count, offset, limit, hasMore, items: [...]}}`.

## Building one

1. Choose the Dashboard artifact type and create it once. Every later write goes to the URL it returns.
2. Read the type's own reference files first (sources, design defaults, features).
3. Run each query in your own session first to confirm arguments and response shape.
4. Write in one batch: dashboard metadata, one dataset per query (`source: {kind: "query", connector: "Filevine", tool: "get_fv_app_Projects", args: {...}}`), and the page.
5. Run each stored call once after writing it. If one fails, fix it and say so.
6. Do not screenshot to verify unless the user asks.

## Page rules

- No network access, modules or React. `d3` v7 is a global.
- No hard-coded numbers. Use `—` placeholders with `data-source`, `data-field`, `data-row`.
- Draw everything inside `dash.onData(fn)`. Handle `ok`, `loading`, `connect` and `error`. Never leave an old chart up.
- Use the `dash-skeleton` class while loading.
- Set data values with `textContent`, never `innerHTML`.
- For chart styling, use a data-visualization skill. This file covers only the Filevine part.

## Loader pattern

Filevine answers are not plain rows, so each dataset needs a loader that unwraps the answer, flattens ID objects, trims dates and follows pages. Loaders only reshape. Count, join and bucket in `dash.calc` so readers can see the formula. Keep IDs as strings and join on the string form.

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

## Dashboards that worked

- Open cases by phase and by project type.
- New cases per month (last 12 months, including zero months).
- Idle cases: under 7, 7 to 30, 30 to 90, over 90 days since last activity.

Untested ideas: open and overdue tasks by user, upcoming deadlines, upcoming appointments. Read each tool's schema first.

Not possible through this connector: settlements, liens, medical specials, A/R aging, anything billing. See SKILL.md section 3.

## Gotchas

- The org ID is fixed in each stored query. A dashboard for another firm needs its own.
- Every viewer needs a Claude account, their own Filevine connection, and access to those cases in Filevine. Otherwise a section shows an error.
- Connector dashboards cannot be shared by public link. Outside guests invited by email can open the page, but connector data does not load for them. For outsiders, make a snapshot copy that stores the rows as a file.
- Sharing is done by the owner from the Share menu. Claude cannot change it.
