# How Filevine returns data

Checked 2026-11-06 against a 33-project firm. If a live call disagrees, the live call wins.

- Lists come back as `{content: {count, offset, limit, hasMore, items: [...]}}`. `count` is the size of that page, not the total. Page with `offset += limit` while `hasMore` is true. Do not follow `links.next` or `links.self`.
- IDs are objects like `{"native": 12345, "partner": ...}`. Read `.native`. Keep IDs as strings when joining.
- Dates are ISO UTC strings. The first 10 characters are the day.
- `requestedFields` is a comma-separated string of field names. It keeps answers small and leaves out personal data.
- The project list carries phase name, project type ID, created date and last activity, so no per-case calls are needed for those.
- `excludeArchived: true` on projects removes the Archived phase, so it means open cases.
- Project types: `get_fv_app_ProjectTypes` gives IDs and names. `get_fv_app_ProjectTypes_phases` gives each type's phases (with a position, and some are permanent, such as Archived). `get_fv_app_ProjectTypes_sections` gives each type's sections: a selector, whether it is a form or a collection, and its custom fields with their types (Currency, Text and so on).
- Users: `get_fv_app_Users` needs `requestedFields` that include `user`, because the ID is nested there. That also returns emails.
- Tasks: `get_fv_app_users_tasks` takes a `userId`. It returns `noteId`, `targetDate`, `isCompleted`, `assigneeId`, `projectId` and `body` (free text, may hold client details).
- Results over about 50,000 characters arrive as a file. `requestedFields` avoids that.

Not yet seen: row shapes for deadlines and appointments (the test project had none), populated forms and collections, contacts, notes, documents, and firms with more than 1,000 projects.
