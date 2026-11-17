# Local Hermes reporting: consent and recovery

For Room totals and binding, use [Room usage](./room-usage.md). Local status
is not proof that a Room has received usage.

## Agent-safe reads

- `artifactbridge telemetry hermes profiles --json` discovers local stores.
  It reads compatibility metadata, not usage. It does not sign in, create a
  ledger or start a service. Preserve unsupported and unreadable rows. An
  incomplete scan returns `discovery_failed`; it is not an empty list.
- `artifactbridge telemetry status --json` reads consent, destination,
  selected profiles, revision, acknowledged usage time, queue and service
  evidence. It does not refresh credentials, migrate state or retry revocation.
  `local_state_unavailable` means the read failed, not Off or zero usage.
- Compatible local status has `hermesDesktopApi: 1`, `ownerProtocol: 1` and
  `hermes.protocol: 1`. Do not infer compatibility from a version string.
  Unknown or missing contracts require an update or a fresh successful read.

Paths and profile labels stay local. Do not copy them into Room usage reports.
A null time is unknown. Hermes acknowledgement time is separate from the
legacy relay's upload time and remains visible after disable.

## Human consent boundary

A person runs `artifactbridge telemetry enable --clients hermes` to choose
profiles and confirm each role, including Unknown role. Sign-in is a separate
human action. Preserve the selected workspace and credential authority.
Never run login, borrow another credential or remove an agent marker to make
reporting enablement succeed.

The person's desktop can submit the confirmed selection through
`artifactbridge telemetry enable --clients hermes --request-stdin --json`.
The CLI help and [public CLI reference](https://www.artifactbridge.com/docs/cli#commands-telemetry)
define the bounded schema-1 request, exact discovery handles, expected revision
and explicit roles. Renew resubmits the complete mapping. Change profiles
replaces the Hermes set and preserves other selected clients. Changes affect
new eligible sessions, not historical usage.

Agents do not submit consent, even when they can access unmarked human OAuth.
The same bytes under that OAuth cannot distinguish a script from a desktop
confirmation. `confirmed: true` is acknowledgement, not an authority token.
Marked agent OAuth and selected agent tokens receive exit 3 and
`human_consent_required` before mutation or refresh. Relay the CLI's human
recovery command instead of replacing the authority.

## Recovery

- `workspace_changed`, `consent_changed` or `profiles_changed`: refresh the
  relevant reads. A person must confirm the new snapshot. Do not replay stale
  selections or accept suggested roles as confirmation.
- `provisioning_outcome_unknown`: the remote grant may exist. Do not replay
  enable. Refresh status and ask the person to reconcile the grant. A known
  remote grant with failed local persistence also requires reconciliation.
- A saved grant with `service_start_failed`: read status. A person can run
  `artifactbridge telemetry hermes restart --json` under valid, unexpired
  consent. Restart changes no grant, revision or mapping. A saved start error
  is not cleared by status polling or a contradictory running-manager result.
- A person runs `artifactbridge telemetry disable --json` to stop the whole
  relay, including non-Hermes clients. Local Off is durable before remote
  revocation. Pending IDs survive failure. A later explicit disable retries
  those IDs, even without an active config; polling never retries them.
- For update instructions, open
  <https://www.artifactbridge.com/docs/cli#commands-update>. The link performs
  no installation, update, login or reporting consent.
