# Factory transport

The factory surface is disabled until an operator enables a qualified internal
binding. There is no self-enrollment tool or UI. Do not try to enable it with a
source report or a Room post. All four tools deny absent, disabled or unqualified
bindings. The coordinator remains denied in this unit.

1. Discover with `artifactbridge_list_factory_actions`, using your own supported
   member credential and current workspace. If denied, hand setup to the operator.
   Never borrow the reporting owner's credential.
2. `artifactbridge_request_factory_action` takes `request` with one stable UUID
   `request_id`, the fixed open `room_id`, `notice_version` from discovery and
   strict `action`. Unknown fields and operations fail validation.
   - `start`: immutable Linear `issue_id` UUID, exact display `issue_key`, `scope`.
   - `answer`: exact `question_event_id`, `ask_id`, `ask_revision`, `work_id`,
     `attempt_id`, `expected_work_revision`, and `response` with one exact
     `choice_key` or unchanged `text`. Read the published grammar first.
     Pass your owned joined `actor_participant_id`; web humans pass null.
     Only persisted ordinary-factory questions under reviewed server policy
     permit capture. A Room answer uses the same transaction and permission rule.
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

## Questions and answers

The operator provisions reviewed ask-policy entries with the binding, keyed by
its exact qualified manifest digest. A publisher references a policy key; it
cannot create or change answer authority. Unknown, missing or corrupt policies
remain human-only. Credential/setup, approval, consent and unclassified gates
stay on their existing authorized-human surface. Do not relay typed approval.

A saved Room answer resolves that Room question and captures pending transport
atomically. It is not engine acceptance or proof that a worker resumed.
Read its `factory_request_id` with `artifactbridge_read_factory_request`.
The receipt retains the exact response and answer/question event IDs.
`answer.status` distinguishes pending, accepted, rejected and unconfirmed.
Rejected answers carry `answered_elsewhere` (safe winning key), `ask_superseded`,
or `validation_failed`. Validation rejection links `correction_question_id`;
the old Room question stays resolved. Read the correction before answering.
Unconfirmed delivery requires reconciliation, never a new answer identity.


## Reporting-owner adapter

Only the configured outside-source owner's supported credential can call
`artifactbridge_exchange_factory_bridge`. This credential remains member-wide,
not source-only; keep it out of the coordinator. Requester identity is separate.
`exchange` supports `pull` with `limit` ≤20 and optional exact `request_ids`,
`ack` with ≤20 `outcomes`, one `publish`, or one `settle`.
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
Acknowledgment outcomes also return `correction_question_id` when present,
including on an exact acknowledgment replay. Retain it with the engine receipt.

`publish.publication` carries stable `engine_event_id`, fixed `room_id`,
owned joined `actor_participant_id`, `work_id`, `attempt_id`, `work_revision`,
and strict `content`:
- `kind:question`: ask identity/revision, prompt, decision, intended member,
  verified routing reason, policy key/digest, grammar and reviewed gate URL.
- `kind:result`: summary, produced URLs, checks, limits and next action.
  Use one event identity per terminal outcome, not each snapshot revision.

Append and correlation commit together. Exact replay returns the same event;
changed content conflicts, including a changed body at an equal ask revision.
Events preserve passive human Room/Inbox attention but never wake agents,
launch workflows or deliver execution webhooks. Ordinary outside-work reports
stay passive and do not publish factory questions.

`settle` names the exact question/ask/revision, owner participant, stable
engine-event identity, `reason_code` (`answered_elsewhere` or `ask_superseded`),
reason and nullable safe winning key. It uses asking-owner withdrawal and Inbox
settlement in the correlation transaction. An already answered or withdrawn
question keeps that lifecycle; no human answer is forged and nothing reopens.
