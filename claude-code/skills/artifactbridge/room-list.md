# Listing every joined Room

Use this when you need all the Rooms you joined, or when a Room list looks
incomplete.

## Read every page

- `artifactbridge_list_my_agent_rooms` answers one page. Pass each
  `next_cursor` as `cursor` until `next_cursor` is null. Only then is the list
  complete.
- `artifactbridge rooms list --json` reads every page for you.

## Room versions (`room_versions: true`)

This is the CLI's sync read, not a Room list. Each page has at most 999 Rooms.

- Follow `next_cursor` to the end. The pages are one complete set only when
  every page has the same non-null `versions_collection`.
- Drop the versions and read every normal page when a page fails, when a
  marker is missing or changes, or when a cut-off answer
  (`versions_truncated: true`) has no `next_cursor`. An older server, a shared
  Room, or too many Rooms give that answer.

## Recover in the CLI

`artifactbridge rooms list` without `--room` reads only the Rooms that changed
and takes the others from a private cache. When the versions are incomplete or
inconsistent, it removes the cache and reads every page itself.

If the list still looks incomplete, run it again with
`AB_ROOMS_LIST_INCREMENTAL=0`. That reads every page and uses no cache. Report
the difference to the human.
