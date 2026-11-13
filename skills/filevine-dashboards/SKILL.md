---
name: filevine-dashboards
description: Use when building a live dashboard or chart from Filevine data (cases by phase, open cases, workload), or when asked how a Filevine dashboard is shared or kept current.
license: Apache-2.0
---

# Filevine dashboards in Claude

Use the Dashboard artifact type. It gives you its own instructions when you create one, and they win over this file. This file covers only what is specific to Filevine.

## Live or snapshot

Default to live: it stores queries and calls Filevine on every open, through each reader's own connection. Make a snapshot (rows stored as a dated file) only when the person wants numbers that stay "as of" a date, or the readers have no Filevine access. Say plainly that a snapshot is not live.

A snapshot file is readable by everyone who can open the dashboard, whatever their Filevine access. Pull it with `requestedFields`, aggregate to what the page shows, and leave out client names unless asked.

## Queries

Only `get_*` tools. Never put a write tool in a stored query. Source form: `{kind: "call", connector: "Filevine", tool, args}`.

```json
{
  "header_x_fv_orgid": 0,
  "excludeArchived": true,
  "limit": 1000,
  "offset": 0,
  "requestedFields": "projectId,phaseName,projectTypeId,createdDate,lastActivity"
}
```

That is `get_fv_app_Projects` and returns open cases with phase, type and last activity, so no per-case calls are needed. Use the real org ID as a number. `get_fv_app_ProjectTypes` (`limit: 100`, `requestedFields: "projectTypeId,name"`) gives type names to join on `projectTypeId.native`.

Filevine answers are not plain rows, so each dataset needs a loader that unwraps `content.items`, reads `.native`, trims dates and follows pages. Count, join and bucket in `dash.calc`, not in the loader.

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
      if (!page || !Array.isArray(page.items)) throw new Error('Filevine did not return a project list');
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

If a loader stops at its page cap it returns `cut: true`; say so on the page. A very large firm repeats those calls on every open, so prefer a narrower query or a snapshot.

## Per-case and per-person calls

Some tools take one ID (vitals per project, tasks per user). Do not loop over every case. Make the ID a filter the reader chooses, and keep the rule that limits it narrow.

## Sharing

Only the owner shares, from the Share menu. Claude cannot change it. By default each reader needs their own Filevine connection and access to those cases, or a section shows an error. The type lets the owner switch a connector to run on the owner's access. That is the owner's decision; never switch it on, and explain the trade-off first. Connector dashboards do not load data for outside guests; give them a snapshot.

## Not possible

Anything billing. Settlements, liens and medical specials only where the firm keeps them in its own fields.

Basics (connecting, org ID, what Filevine cannot give, response shapes): use the `filevine` skill.
