# Firm model: what a firm's Filevine looks like, kept between chats

A firm model is a short record of how one firm uses Filevine: its project types, each type's phases, each type's sections and field types, and what is not yet known. Claude does not remember it between chats. Store it in an artifact's database so any later chat can read it.

## What goes where

- **In this skill:** how Filevine returns data (`data-shapes.md`), limits, safe habits, this procedure and the page template. Nothing firm-specific.
- **In the artifact:** facts about one firm. IDs, names, counts, field types, gaps.

## Every firm is different

Project types, phases, sections, selectors and field names belong to the firm. Do not assume any exist, and do not reuse one firm's names for another. Only the kinds are common: a section is a form (one record) or a collection (repeating rows), and a field has a type such as Currency, Date, Text, Dropdown or Boolean. Read the firm's own setup every time, and use real names only as examples in a reply, never in the skill.

## Safety

Store structure and counts only. No client names, no case data, no field values, no contact details. The artifact belongs to whoever made it. Say who can open it. If the person wants it shared, they share it from the artifact's Share menu.

## Procedure

1. **Find the artifact.** List the person's artifacts and look for a title that starts with `Filevine firm profile`. Name a new one `Filevine firm profile – <firm name>`; the page template's title is the prefix. The list returns at most 50 per call, so an older artifact can be missing. If none is found, page the list or ask for the link before creating a new one. If two share the title, ask which is current. Do not create a duplicate.
2. **If it exists, read the `model` collection first** and use it. Re-read live only what the person needs or what looks stale (`meta.asOf`).
3. **If not, read the firm.** Get the org ID. Then, with small calls and `requestedFields`: project types; counts per type and phase from the project list (open and archived); phases for each type that has cases; sections for each type that has cases. Do not read forms or collections per project to build the model.
4. **Write the model** to the artifact database as documents in collection `model` (below). Use `ArtifactData` `set`, and `if_version` on any existing document.
5. **Publish the page** from `assets/firm-model-page.html` with the `db` capability (read the artifact-capabilities skill first). The page needs no edits: it reads the `model` collection.
6. **Record gaps.** Anything not read goes in `gaps`. Say what was seen and what was assumed.

## Stored documents (collection `model`)

| Doc | Shape |
|---|---|
| `meta` | `{orgId, firm, asOf, note}` |
| `types` | `{items: [{id, name, open, archived}], note}` |
| `phases` | `{typeName, items: [{name, position, permanent, open}], note}`, one type at a time |
| `sections` | `{typeName, items: [{name, selector, kind: "form" or "collection", fieldCount, types: {Currency: 2, Text: 5}}], note}` |
| `learned` | `{items: [{meaning, where, learnedAt}]}`. Append only when a task actually used something: `meaning` in the firm's words (for example what a field holds), `where` the section selector and field selector, `learnedAt` a date |
| `gaps` | `{items: ["..."]}` |

## Notes from testing

- On a large firm, types, phases, sections and counts need only list calls. Project pages and the section list arrive as files, so count them with a script. A large share of section fields are layout (Header, Instructions, ActionButton, document tools), so count data fields separately if the page should.
- Money often sits in Currency fields and in the vitals call (`get_fv_app_Projects_Vitals`), when the firm has set it up.

- A test or small firm has many near-empty sections. Show them as "none listed" rather than hiding them.
- The model keeps field-type counts, not every field name. Names go into `learned` one at a time, when a task uses them, with what they mean at this firm. Do not bulk-copy a section's fields into the model.
- Money figures exist only where the firm keeps Currency fields. The page lists those.
- Writes from a second chat: a read, merge and update worked, a stale `if_version` was rejected, nothing was overwritten. Whether the first chat saw the change live was not confirmed.
- Not tested: connecting to a second firm, a firm with many types, and the page on a phone.
