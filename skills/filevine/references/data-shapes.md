# How Filevine returns data

Checked 2026-11-06 against two firms, one small and one large. Form and collection shapes were seen on a firm with few filled-in fields. If a live call disagrees, the live call wins.

- Lists come back as `{content: {count, offset, limit, hasMore, items: [...]}}`. `count` is the size of that page, not the total. Page with `offset += limit` while `hasMore` is true. Do not follow `links.next` or `links.self`.
- IDs are objects like `{"native": 12345, "partner": ...}`. Read `.native`. Keep IDs as strings when joining.
- Dates are ISO UTC strings. The first 10 characters are the day.
- `requestedFields` is a comma-separated string of field names. It keeps answers small and leaves out personal data.
- The project list carries phase name, project type ID, created date and last activity, so no per-case calls are needed for those.
- `excludeArchived: true` on projects removes the Archived phase, so it means open cases.
- Project types: `get_fv_app_ProjectTypes` gives IDs and names. `get_fv_app_ProjectTypes_phases` gives each type's phases (with a position, and some are permanent, such as Archived). `get_fv_app_ProjectTypes_sections` gives each type's sections: a selector, whether it is a form or a collection, and its custom fields with their types (Currency, Text and so on).
- Users: `get_fv_app_Users` needs `requestedFields` that include `user`, because the ID is nested there. That also returns emails.
- Tasks: `get_fv_app_users_tasks` takes a `userId`. It returns `noteId`, `targetDate`, `isCompleted`, `assigneeId`, `projectId` and `body` (free text, may hold client details).
- Form values (`get_fv_app_Projects_Forms`): a flat object keyed by field selector. Empty fields are null. A form with no fields defined returns an empty object.
- Collection rows (`get_fv_app_Projects_Collections`): rows under `items`, each with `itemId` (a GUID string), `createdDate`, `itemLink` and `dataObject`. Empty fields are left out of `dataObject`. An empty collection returns count 0.
- Currency comes back as a JSON number with eight decimals.
- Section IDs are plain integers, not native-ID objects. Pass each section selector to the forms and collections tools.
- Results over about 50,000 characters arrive as a file. `requestedFields` avoids that.

- Vitals (`get_fv_app_Projects_Vitals`): one call per project returns the firm's at-a-glance fields as a plain list under `content` (not `content.items`), each `{fieldName, friendlyName, fieldType, value, position}`. Which fields appear is the firm's own setup, and can include key dates, money totals and team roles. Values are strings, empty when unset. It is the cheapest per-case route to money and key dates, but still one call per project.
- Project sections (`get_fv_app_ProjectTypes_sections`): each item has `id`, `sectionSelector`, `isCollection`, `name`, `isProtected` and `customFields[]`. Each custom field has `fieldSelector`, `name`, `customFieldType` (Currency, Text, Date, Boolean, PersonLink, Dropdown, Header, Instructions, ActionButton and others), `dropdownItems`, `behaviors`, `isRequired`. Without `requestedFields` the answer can be hundreds of thousands of characters. Many fields are layout (Header, Instructions, ActionButton), not data.
- Phases (`get_fv_app_ProjectTypes_phases`): `phaseId`, `name`, `isPermanent`, `position`. The list is not in order and positions are not contiguous. Sort by `position`.
- Appointments (`get_fv_app_Projects_Appointments`): `appointmentId`, `projectId`, `calendarEventType`, `title`, `allDay`, `startUtc`, `endUtc`, `notes`, `location`, `ownerId`. A statute-of-limitations date can appear here as a `CustomDeadline` appointment even when the deadlines list is empty.
- Deadlines (`get_fv_app_projects_deadlines`): returned an empty list on every project tried. Row shape still unseen.
- Tasks per project (`get_fv_app_projects_tasks`): same row shape as user tasks. `targetDate` is ISO UTC at midnight.
- Collection rows: `dataObject` came back empty when `requestedFields` named only `itemId,createdDate`. Ask for the fields you want, or omit `requestedFields`, to get row values.
- Users: each item has `user` (with `userId`, `firstName`, `lastName`, `email`, `username`), `userName`, `isActive`. Its `links.next` was malformed.

- Project contacts (`get_fv_app_Projects_contacts`): each item has `roles` (strings written "Section: Role") and `orgContact`, a large nested record with name, email, phones, addresses, job title and links. `requestedFields` works only on top-level keys, so asking for `orgContact` returns all of it, personal details included. Ask for `roles` alone unless the person needs contact details.
- Notes (`get_fv_app_Projects_Notes`): `requestedFields` names that don't exist are silently dropped, so a wrong name gives back only the ID. Check the real field names with one item before relying on them. Note bodies hold free text and client details.
- Folders (`get_fv_app_Folders_list`): flat list with `folderId`, `parentId`, `projectId`, `name`, `isArchived`, `isProtected`. The top folder of a case can be named after the client, so folder names are personal data too.

Not yet seen: row shapes for deadlines and appointments (the test project had none), document contents, deadline rows, and firms with more than 1,000 projects.
