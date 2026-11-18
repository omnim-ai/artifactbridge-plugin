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
- `artifactBridgeCredits`: credits that ArtifactBridge runs used. `status` is
  `final`, `partial` (the figure is not final yet), `no_runs` (no credited
  runs) or `unavailable` (no credit reading; do not estimate one). The Room
  page hides an unavailable credits line in a Room with no Hermes line. This
  read still gives the true status.

## Coverage

Hermes coverage is always partial. `hermes.limitations` lists why. Only the
profiles a person selected report, and only usage after consent counts. Coding
runs outside Hermes, including ArtifactBridge-dispatched container runs, are
not included in the Hermes subtotal. ArtifactBridge credits are separate; the
first release has no authoritative per-Room credits reading. Do not estimate,
convert or add credits to the Hermes USD estimate. Say that the figures are
partial when you report them.

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

`workspaceHasHermesReports` is one yes/no for the whole workspace. It becomes
`true` when the server accepts the first eligible Hermes usage in any Room of
the workspace, also usage with an estimated cost of zero. Setup, consent, a
binding, a baseline, health reports and refused uploads do not set it. It
stays `true` after consent expires, after a person turns reporting off and
after retention removes the usage. It tells that Hermes reported at some
time. It does not tell that reporting is on now. It can be `true` because of
a Room that you cannot read; the summary tells nothing more about that Room.

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

A person or an agent may bind a session to a Room. Binding does not turn
reporting on:

- `artifactbridge telemetry bind --client hermes --profile NAME --session ID --room REF --json`
  binds one session of a selected profile to a Room. Bind before the session
  uses a model; usage from before the binding is not reported.
  `binding_conflict` means that the session already has another Room. Start a
  new session for this Room.

A person decides these. Do not run them for the person:

- `artifactbridge telemetry enable --clients hermes` turns reporting on or
  renews it. On a computer without a browser, the person first signs in with
  `artifactbridge login --no-browser --callback-port PORT` over SSH.
  Consent expires after 30 days, and reporting pauses until the person renews.
  Hermes keeps working. Usage from the paused time is not reported later.
- `artifactbridge telemetry hermes restart` starts the reporting service
  again under the current consent. It does not renew expired consent. When
  `service.manager` is `external`, the person's supervisor runs
  `artifactbridge telemetry relay run`, and restart only returns that
  command with `service_managed_externally`.
- `artifactbridge telemetry disable` turns reporting off on this computer
  first. When `serverRevokePending` is `true`, the person runs it again later
  to retry the saved revocations.
- `artifactbridge telemetry remove` removes the local setup.

With an agent credential, these commands stop with exit 3 and
`human_consent_required`, and change nothing. Give the person the
`recovery.command` from the answer, then stop.
