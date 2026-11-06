# Firm model

A short record of how one firm uses Filevine: its project types, each type's phases, sections and field types. Claude does not remember it between chats, so keep it in an artifact's database.

Project types, phases, sections and field names belong to the firm. Never assume them or reuse one firm's names for another. Read the firm's own setup. Store structure and counts only: no client names, no case data, no field values, no contact details.

## Where it lives

One artifact titled `Filevine Models`, holding every firm and every project type the person works with. Not one artifact per firm.

1. Look for it by title. The list returns at most 50, so page it before concluding there is none.
2. If there is none, create it, then look again. If two now exist because another chat did the same, keep the oldest, copy across anything only the new one has, and ask the user before deleting the extra.
3. If the firm is already in it, read that first and re-read live only what the task needs or what looks stale.
4. If not, learn the firm with small calls and `requestedFields`: org ID, project types, counts per type and phase, phases and sections for each type that has cases. Do not read forms or collections per project. On a large firm, count with a script.
5. Write with `ArtifactData` `set` or `batch`, using `if_version` on existing documents. Never write one type's facts into another type's document.
6. Publish the page from `assets/firm-model-page.html` with the `db` capability. It reads the `model` collection and needs no edits.

## Documents (collection `model`)

| Doc ID | Shape |
|---|---|
| `org_<orgId>` | `{orgId, firm, asOf, note}` |
| `type_<orgId>_<typeId>` | `{orgId, id, name, open, archived, note, phases: [{name, position, permanent, open}], sections: [{name, selector, kind: "form" or "collection", fieldCount, types: {Currency: 2, Text: 5}}]}` |
| `learned_<orgId>` | `{items: [{meaning, where, learnedAt}]}`. Add an entry only when a task actually used something: what a field means at this firm, where it is, the date. |
| `gaps_<orgId>` | `{items: ["..."]}`. Anything not read, and anything assumed. |

Many section fields are layout (Header, Instructions, ActionButton), not data. Keep field-type counts, not every field name. Money lives in Currency fields and in the vitals call, only where the firm set them up.
