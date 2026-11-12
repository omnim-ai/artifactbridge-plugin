# Room usage

Use this when a person asks how much a Room's agents used, or what they cost.

`artifactbridge_read_room_usage` returns the same summary as the Room page.
Any member who can read the Room can read it. It writes nothing.

## Sections

The summary has three separate sections. They measure different things. Never
add them into one total, and never convert one into another.

- `hermes`: token counts and estimated cost that members' Hermes setups
  reported for this Room in the last 90 days (`retainedWindow`). Each figure
  has a `value` and an `availability`: `available`, `partial` (some reports
  did not measure it) or `unavailable`. A `null` value is unknown, not zero.
  `estimatedCostUsd` is an exact decimal string in US dollars. It is a client
  estimate, not a bill.
- `otherClients`: usage that other agent clients (for example Claude Code or
  Codex) sent. `null` when none sent usage for this Room.
- `artifactBridgeCredits`: credits that ArtifactBridge runs used. When
  `status` is `unavailable`, there is no credit reading. Do not estimate one.

## Coverage

Hermes coverage is always partial. `hermes.limitations` lists why. For
example, only the profiles a member selected report, only usage after consent
counts, and work outside Hermes coding is not included. Say that the figures
are partial when you report them.

## No Hermes reports

When `hermes.status` is `no_reports`:

- If `workspaceHasHermesReports` is `false`, the result has a `note`. Give the
  person that note as written: "No retained Hermes usage is reported for this
  Room. The workspace has not accepted eligible Hermes usage. This does not
  establish consent or setup state."
- If it is `true`, the workspace has Hermes reports, but this Room has no
  retained Hermes usage. Do not guess which members or Rooms reported.

## Errors

- `not_found`: you cannot read this Room.
- `unavailable`: the read failed. Try again later. Do not report zero usage.
