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

## Known gaps

`hermes.knownGapSetupCount` is the number of setups that reported a time when
they could not report usage. `hermes.knownGaps` has at most two groups: setups
whose latest gap has a reported end, and setups whose latest gap has none.
Each group gives `setupCount` and the `startedAt` and `endedAt` of its latest
gap. Report the groups. Do not name members or setups.

- `[]` and `0` mean that no setup reported a gap in the last 90 days.
- `null` with `known_gaps_unavailable` in `hermes.limitations` means unknown,
  not zero.
- `endedAt: null` means that no end was reported. It does not mean that the
  setup still cannot report. The summary does not give the current state.
- `delayedSetupCount` is always `null`. Do not estimate delays.

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

## Local Hermes reporting

These commands read the reporting setup on this computer. They do not read
usage or change anything:

- `artifactbridge telemetry status --json` gives `cliVersion`, `authority`
  (`kind` is `human`, `agent`, `signed_out` or `unknown`) and a `hermes`
  block: `state`, `consentRevision`, `profiles`, `workspaceId`,
  `lastCollectedAt`, `lastReportAt`, `queueDepth`, `serverRevokePending` and
  `service.lastError`. `workspaceName` is always `null`.
- `artifactbridge telemetry hermes profiles --json` lists the Hermes profiles
  on this computer. When `supported` is `false`, `supportReason` tells why.

A person decides these. Do not run them for the person:

- `artifactbridge telemetry enable --clients hermes` turns reporting on or
  renews it.
- `artifactbridge telemetry hermes restart` starts the reporting service
  again under the current consent. It does not renew expired consent.
- `artifactbridge telemetry disable` turns reporting off on this computer
  first. When `serverRevokePending` is `true`, the person runs it again later
  to retry the saved revocations.
- `artifactbridge telemetry remove` removes the local setup.

With an agent credential, these commands stop with exit 3 and
`human_consent_required`, and change nothing. Give the person the
`recovery.command` from the answer, then stop.
