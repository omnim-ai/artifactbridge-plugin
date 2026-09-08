# Selection queries for a vault import

A selection query picks the notes to import. Pass the query as the `select`
option to `artifactbridge_plan_document_import`. The server builds the full
plan and reports the imported notes and the deselected notes. Derive the match
count from the plan, as "Preview first" describes below, and show that count to
the human.

The query never reads a file or the network. The query never selects a note in
`.obsidian/` or `.trash/`. The query does not expand links. A linked note stays
unselected unless the query selects it. The import reports the unresolved link.

## Predicates

- `path:VALUE` — match a folder or a note path. A plain value matches the path
  and every path under it. Use `*` for one path segment, `**` for any depth,
  and `?` for one character.
- `tag:VALUE` — match a tag. A parent tag also matches its child tags. Write
  the tag with or without a leading `#`.
- `[NAME]` — match a note that has the frontmatter property `NAME`.
- `[NAME:VALUE]` — match a note whose property `NAME` equals `VALUE`.
- `modified-after:YYYY-MM-DD` — match a note changed strictly after
  00:00:00 UTC on that date.
- A bare term — match a note whose body contains the term. The match reads the
  complete body, including fenced code blocks.

All text comparisons ignore case.

## Operators

- `AND` — both sides must match.
- `OR` — either side must match.
- `-` before a predicate — the predicate must not match.
- Double quotes keep spaces. Examples: `path:"Meeting Notes"`,
  `["due date":"next week"]`, and `"launch sequence"`. Do not put a space
  inside the brackets of a property predicate.

Combine two predicates with an explicit `AND` or `OR`. There is no implicit
operator. Two predicates without an operator are an invalid query.

## Examples

- `path:"Product Launch"` — every note in the "Product Launch" folder.
- `tag:launch AND modified-after:2026-01-01` — launch notes changed this year.
- `[status:active] AND -tag:archive` — active notes that are not archived.
- `path:Projects OR path:Areas` — notes in either top folder.

## Preview first

1. Send the query as `select` to `artifactbridge_plan_document_import`.
2. Read the plan. The imported notes appear as document actions. The notes the
   query dropped appear as skipped files with the reason `deselected`.
3. Show the human the match count: the number of imported notes, and the number
   of deselected notes.
4. When the count looks wrong, ask for a new query. Do not apply.

An invalid query returns an `invalid_argument` error. A `modified-after` query
is not yet supported. Report the error to the human and ask for a new query.
