# Factory transport

The factory surface is disabled until an operator enables a qualified internal
binding. There is no self-enrollment tool or UI. Do not try to enable it with a
source report or a Room post. All four tools deny absent, disabled or unqualified
bindings. The coordinator is denied in this unit. More operations arrive in later units.

1. Discover with `artifactbridge_list_factory_actions`, using your own supported
   member credential and current workspace. If denied, hand setup to the operator.
   Never borrow the reporting owner's credential.
2. `artifactbridge_request_factory_action` takes `request` with one stable UUID
   `request_id`, the fixed open `room_id`, `notice_version` from discovery and
   strict `action`. Unknown fields and operations fail validation.
   - `start`: immutable Linear `issue_id` UUID, exact display `issue_key`, `scope`.
   - `guidance`: `work_id`, `expected_work_revision`, `target` containing either
     `attempt_id` or `next_safe_point:true`, and `text`.
   - `inspect`: `work_id`; read-only engine state through the same transport.
   - `stop`: `work_id`, exact `attempt_id` (null for queued/no attempt),
     `expected_work_revision`, and completed `inspection_request_id`. Inspect
     first; stale targets fail. Never silently stop a newer attempt.
   - `retry`: `work_id`, terminal `previous_attempt_id`, `expected_work_revision`,
     unchanged `scope_digest`. The engine rechecks admission and terminal state.
   Text is unchanged, at most 8192 UTF-8 bytes. Human approvals stay on their
   existing human surfaces; scope/review/production-policy bypass is not allowed.
3. `submitted` means saved for transport, not engine accepted or running.
   Recover with `artifactbridge_read_factory_request` and exact `request_id`.
   Same ID/payload replays; different payload conflicts. Foreign/inaccessible
   reads disclose no metadata. Read before any replay; never invent another ID
   because an outcome is unknown. Use operator reconciliation when uncertain.
4. Start receipts include `existing_live_work`: true means nothing new started.
   `task_url` can be null; retain request/work identity instead of inventing a link.
   Guidance received/applied/deferred/rejected and Stop requested/stopped/unknown
   are distinct. Confirm stopped only with engine fence plus exact readback.
   Stop response and leaving chat do not cancel factory work or remove outputs.

## Reporting-owner adapter

Only the configured outside-source owner's supported credential can call
`artifactbridge_exchange_factory_bridge`. This credential remains member-wide,
not source-only; keep it out of the coordinator. Requester identity is separate.
`exchange` is `{"operation":"pull","limit":20}` or `ack` with ≤20 `outcomes`.
Persist pulled intents before effects. Reconcile the engine by stable identity
before redelivery. A 60-second claim expiry permits the same intent, not new work.
For locally recovered, unadmitted intents, pull exact `request_ids` again to
obtain fresh authorization before admission. Removed members or revoked origin
credentials cannot be revived by the reporting owner.

Each acknowledgment supplies `request_id`, `payload_digest`, `claim_id` and the
full revisioned `receipt` from the engine. Preserve `updated_at` on exact retries.
Per-item outcomes include applied, duplicate, stale, conflict and refused.
Equal revision with changed content conflicts. Keep failed rows for reconciliation;
a batch is not an all-items success receipt. Closed Rooms pause delivery until
an authorized reopen. Reporting failure is not execution failure.
