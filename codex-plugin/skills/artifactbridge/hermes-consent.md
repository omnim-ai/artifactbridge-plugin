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

## Workstream registration

An authorized coordinator or integration sets the Room of a Hermes Kanban
workstream once, when the initiative starts. A workstream is one tenant on one
board in one Kanban database. Registration records intent only. It does not
turn reporting on, select profiles, confirm roles, bind a session or change
consent.

- Read first: `artifactbridge telemetry hermes workstreams list --json`. It
  reads only local files. Each entry has `workstream` (`board`, `kanbanDb`,
  `tenant`), `registration` (`live` or `removed`), `roomId`, `revision`,
  `memberId`, `workspaceId` and `state`. A removed entry keeps its owner and
  revision; its `roomId` and `state` are null. A removed entry is not active.
- Set: `artifactbridge telemetry hermes workstreams set --board BOARD
  --kanban-db PATH --tenant TENANT --room UUID --expected-revision N --json`.
  Give the exact board slug, the canonical absolute path of the Kanban
  database file, the exact tenant and the lowercase Room UUID. Do not search a
  home folder for a board; ask the person which board the initiative uses. The
  CLI does not repair or normalize a value. Use `--expected-revision 0` only
  for a workstream that never had a Room, and the listed revision for a
  change. Your credential must be able to read the Room. Then list again and
  check the key, Room and revision. A set saves intent only: automatic
  reporting is not active.
- Remove: `artifactbridge telemetry hermes workstreams remove --board BOARD
  --kanban-db PATH --tenant TENANT --expected-revision N --json`. Only the
  member and workspace that set it can remove it. Sessions that started
  earlier keep their Room. The removal keeps its `revision` and its owner. To
  set a Room again, the same member and workspace use that revision, not 0.
- `stale_revision` (exit 1): list again and use the current revision. For a
  removed workstream, use its listed revision; the message also names it.
  `selected_token_unavailable` (exit 2): your selected token variable is not
  set or is empty, so nothing was sent and the saved sign-in was not used;
  set the variable, then try again.
  `workstream_owned_elsewhere` (exit 3): another member or workspace set it;
  do not remove it. `room_access_denied` (exit 3): use a Room that you can
  read. `kanban_db_not_found` or `kanban_db_not_canonical` (exit 2): give the
  real canonical path. `not_signed_in` (exit 2): ask the person to sign in.
  `agent_account_unavailable` (exit 2): your agent runtime has no signed-in
  account, so nothing was sent; follow the message, then try again.
  `account_check_unavailable` or `ledger_unavailable` (exit 1): nothing
  changed; try again once.
- `state` is `waiting_for_consent` until a person turns Hermes reporting on,
  `workspace_mismatch` when reporting goes to another workspace, and
  `kanban_db_missing` or `kanban_db_replaced` when the database file is gone
  or is another file; set it again for a replaced file. Automatic card binding
  is not available yet: the best state is `census_pending`, never `active`.
  Do not tell anyone that card usage reports.

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
- `provisioning_outcome_unknown`: the remote grant may exist. Read status. If
  Hermes is not On, the person runs
  `artifactbridge telemetry enable --clients hermes` again.
  `local_state_unavailable` after a remote grant: read status. If status
  works, note its `clients`; the person runs
  `artifactbridge telemetry disable --json`, then
  `artifactbridge telemetry enable --clients <those clients>,hermes`. If
  status also returns `local_state_unavailable`, the local reporting files
  cannot be read; the person makes them readable first.
- A saved grant with `service_start_failed`: read status. A person can run
  `artifactbridge telemetry hermes restart --json` under valid, unexpired
  consent. Restart changes no grant, revision or mapping. A saved start error
  is not cleared by status polling or a contradictory running-manager result.
- `hermes.service.manager` is `external`: the person's supervisor (for example
  supervisord, Docker or runit) runs `artifactbridge telemetry relay run` as
  the same user with the same state folder. `retryAllowed` is `false`, and
  `telemetry hermes restart` returns `service_managed_externally` (exit 2)
  with the `supervisor.command`. `state` is `running` only while the recorded
  relay holds the setup's owner lock. Give the person the command; do not
  start a relay or edit a supervisor yourself. After a renewal the relay
  exits; the supervisor starts it again. The person registers the entry. A
  supervisord example for the user `me`:

  ```ini
  [program:artifactbridge-relay]
  command=/home/me/.local/bin/artifactbridge telemetry relay run
  user=me
  environment=HOME="/home/me"
  autorestart=unexpected
  exitcodes=0
  ```

  With runit or Docker, run the same command in the foreground as the same
  user with the same `HOME`, and restart it only on failure.
  `artifactbridge help telemetry relay run` prints runit and Docker examples.
- `service_manager_conflict` (exit 2): the setup uses another service manager.
  The person runs `telemetry disable`, removes the old service or supervisor
  entry, checks that the relay stopped, then enables with the new
  `--service-manager`. Nothing was changed.
- A person runs `artifactbridge telemetry disable --json` to stop the whole
  relay, including non-Hermes clients. Local Off is durable before remote
  revocation. Pending IDs survive failure. A later explicit disable retries
  those IDs, even without an active config; polling never retries them.
- For update instructions, open
  <https://www.artifactbridge.com/docs/cli#commands-update>. The link performs
  no installation, update, login or reporting consent.
