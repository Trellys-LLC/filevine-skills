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

One artifact named `Filevine Models` holds every firm (org) the person works with, and every project type within each firm. Do not make one artifact per firm.

1. **Find it.** List the person's artifacts and look for the title `Filevine Models`. The list returns at most 50 per call, so an older artifact can be missing: page the list before concluding there is none.
2. **Create it without making a duplicate.** If none was found: create one, then search again. If two now exist, keep the oldest (by creation time), put anything you wrote into it, and delete the one you just created. Another chat may be doing the same at the same moment.
3. **If the firm is already in it, read first** (`org_<orgId>` and its `type_` documents) and use that. Re-read live only what the task needs or what looks stale (`asOf`).
4. **If not, read the firm.** Get the org ID. Then, with small calls and `requestedFields`: project types; counts per type and phase from the project list (open and archived), counted with a script on a large firm; phases for each type that has cases; sections for each type that has cases. Do not read forms or collections per project to build the model.
5. **Write the model** with `ArtifactData` `set` (or `batch`), using the documents below. Use `if_version` on any existing document. Never write one type's information into another type's document.
6. **Publish the page** from `assets/firm-model-page.html` with the `db` capability (read the artifact-capabilities skill first). Its title is `Filevine Models`. The page needs no edits: it reads the `model` collection and shows each firm and each type.
7. **Record gaps.** Anything not read goes in the firm's `gaps`. Say what was seen and what was assumed.

## Stored documents (collection `model`)

Document IDs carry the org and type, so firms and types never overwrite each other.

| Doc ID | Shape |
|---|---|
| `org_<orgId>` | `{orgId, firm, asOf, note}` |
| `type_<orgId>_<typeId>` | `{orgId, id, name, open, archived, note, phases: [{name, position, permanent, open}], sections: [{name, selector, kind: "form" or "collection", fieldCount, types: {Currency: 2, Text: 5}}]}` |
| `learned_<orgId>` | `{items: [{meaning, where, learnedAt}]}`. Append only when a task actually used something: `meaning` in the firm's words (for example what a field holds), `where` the section selector and field selector, `learnedAt` a date |
| `gaps_<orgId>` | `{items: ["..."]}` |

## Notes from testing

- On a large firm, types, phases, sections and counts need only list calls. Project pages and the section list arrive as files, so count them with a script. A large share of section fields are layout (Header, Instructions, ActionButton, document tools), so count data fields separately if the page should.
- Money often sits in Currency fields and in the vitals call (`get_fv_app_Projects_Vitals`), when the firm has set it up.

- A test or small firm has many near-empty sections. Show them as "none listed" rather than hiding them.
- The model keeps field-type counts, not every field name. Names go into `learned` one at a time, when a task uses them, with what they mean at this firm. Do not bulk-copy a section's fields into the model.
- Money figures exist only where the firm keeps Currency fields. The page lists those.
- Writes from a second chat: a read, merge and update worked, a stale `if_version` was rejected, nothing was overwritten. Whether the first chat saw the change live was not confirmed.
- Not tested: connecting to a second firm, a firm with many types, and the page on a phone.
