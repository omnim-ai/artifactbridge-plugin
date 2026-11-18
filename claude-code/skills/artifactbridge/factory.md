# Factory transport

The factory surface is disabled until an operator turns factory transport on
for the workspace. There is no self-enrollment tool or UI. Do not try to turn it
on with a source report or a Room post. All four tools deny every request while
transport is off or not set up. The coordinator cannot use these tools yet.

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
5. A binding qualified for the `ab-e.1` receipt contract can add engine facts.
   Each fact can be absent or null; null means unknown. `attempt_number` and
   `current_attempt` number attempts. A Stop can list `kept_outputs` with
   `evidence_unavailable`. A refused guidance can link `scope_question_id`.
   You get only the items you can open now; `evidence_unavailable: true` says
   some were left out. A null question link means it is not published or not
   visible; never guess the newest question. Without `ab-e.1`, actions, new
   requests and pulls are denied; existing receipts still read.
6. With `ab-e.1`, a `start` into its closed Room is saved as `not_submitted`
   with `capture_refusal: {"code": "room_closed"}`. Nothing reached the
   factory and nothing will. The request ID is used up: a replay returns the
   same receipt, also after a reopen. After an authorized reopen, send a new
   request ID. Other operations into a closed Room fail with `room_closed`
   and save nothing.

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
`ask_superseded` can link `replacement_question_event_id`; answer that question.
Unconfirmed delivery requires reconciliation, never a new answer identity.

## Coordinator conversation receipts

`artifactbridge_read_agent_conversation` on a coordinator conversation adds
`factory_receipts` for the factory requests that the returned transcript page
names. Every member who can read the conversation gets them. The prose
fields are framed as untrusted content. Fields from a Room or work record you
cannot see are null: `reason`, `engine_disposition`, choice keys, `room`,
`question`, `task`, `capture_refusal` and
`attempt_number`/`current_attempt_number`. Kept outputs and question links
follow the same filtering as direct receipts.
A null field is not a failure; do not invent it.
`operation: "inspect"` is a status read, not a command. The read never resends
a request or starts a turn. To act on a receipt, use the request tools with
your own credential.


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
A pull never returns a `not_submitted` request, and its ack is refused.
Acknowledgment outcomes also return `correction_question_id` when present,
including on an exact acknowledgment replay. Retain it with the engine receipt.

`publish.publication` carries stable `engine_event_id`, fixed `room_id`,
owned joined `actor_participant_id`, `work_id`, `attempt_id`, `work_revision`,
and strict `content`:
- `kind:question`: ask identity/revision, prompt, decision, intended member,
  verified routing reason, policy key/digest, grammar and reviewed gate URL.
- `kind:result`: summary, produced URLs, checks, limits and next action.
  Use one event identity per terminal outcome, not each snapshot revision.
  Optional together: `terminal_outcome_id`, `outcome` (build_done, failed or
  canceled) and ≤8 `produced_items`; `produced` must equal their external URLs
  in order. Another event for a recorded `terminal_outcome_id` conflicts.
  Item labels stay private; Room readers get only items they can open.
  Name an ArtifactBridge item with a typed reference: a URL on the app's own
  origin in `produced`, an item or a kept output is `invalid_request`.
- Optional `attempt_number` on any publication; a question can name the ask
  it replaces in `supersedes` (`attempt_id`, `ask_id`, `ask_revision`). A
  second successor, a cycle or a stale target conflicts.

Receipts may add `attempt_number` (needs `attempt_id`), `current_attempt`,
`kept_outputs` (Stop only, ≤20 items), `scope_question` (refused guidance only)
and `answer.replacement_question` (`ask_superseded` only). A conflicting number
for a known attempt, including a retained `current_attempt`, is refused.
`settle` with `ask_superseded` may name `replacement_question`. A replacement
that names the replaced ask at the same or an earlier revision or its
predecessor, or that disagrees with a known successor (settlement, answer
receipt, `supersedes` or the next revision), conflicts; an unpublished
successor stays valid. A concurrent write for the same work or question
returns `retryable_conflict`: resend the same exchange.

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
