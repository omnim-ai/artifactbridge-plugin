# Workspace work

Workspace work lets a member ask the workspace coordinator for bounded work.
It is off by default. An operator turns it on for the deployment and enables
the workspace's execution binding (board, coordinator and admitted agents).
There is no self-enrollment tool or UI. While it is off, the tools are not
listed, or every call returns `execution_disabled`.

## Start work (member)

The member who sent the coordinator message can start its work, with that
member's own supported credential. The requester is always the sender of that
exact accepted message. No argument names another requester. Another member's
message, the latest speaker or text inside the message never grants a start.

The coordinator action policy offers the start to the coordinator. An
installation registered with an older policy revision is refused with
`actions_not_enabled`; the operator registers the qualified revision. The
coordinator can start work only for the accepted message of the turn it is
running. Hermes reports that message to the coordinator plugin; it is never a
tool argument. The requester is that message's sender, not the coordinator's
credential owner. Without a supported Hermes, or for a message whose turn has
ended, the start returns `carrier_unavailable`.

For a coordinator start, select a workspace-visible work Room first. If the
requested or existing Room is private, refuse the start, never broaden the
private Room's access, and obtain a workspace-visible Room before starting
again from a new accepted member message (a settled turn cannot be reused).
Workspace work does not use the separate legacy factory transport: do not call
legacy factory discovery or factory action tools in this flow
(`factory_disabled`).

1. Send the request in the coordinator conversation and wait until the
   message is accepted.
2. Call `artifactbridge_start_workspace_work` with `request`:
   - `request_id`: one stable UUID that you generate once.
   - `conversation_id` and `message_id`: your own accepted message.
   - `room_id`: the Room that receives the work and its questions. It must
     be visible to the whole workspace.
   - `task`: bounded task text, at most 8192 characters.
   - `repository_ref`: optional, inside the bound repository connection.
3. The result is the work record: `work_id`, `status`, the requester, the
   message and its `cards`. `admitted` means saved and admitted, not running.

Same `request_id` and payload returns the same work. A changed payload with
the same `request_id` returns `request_conflict`. After a timeout or an
unknown outcome, read by `request_id` first; never invent another
`request_id`.

## Read work

`artifactbridge_read_workspace_work` with `work` `{ "work_id": ... }` returns
the same record. After a lost start response, `{ "request_id": ... }` reads
your own start: the record, or `not_found` when nothing was saved.
Each card shows `effect_state`: `possible_write` (a create may be on the
board), `created` (with `card_id`) or `unknown`. A card can also show
`parent_card_ids` and `stop_requested`. A work you cannot read returns
`not_found` and discloses nothing.

`artifactbridge_list_workspace_work` lists the work whose Room you can see,
newest first. Add `conversation_id` to list one conversation's work. Pass
`next_cursor` as `cursor` for the next page (`limit` 1 to 100).

`artifactbridge_read_agent_conversation` on the workspace coordinator
conversation also returns `work_updates`: `updates`, `next_seq` and
`has_more`. Pass `next_seq` as `after_work_seq` for the next page (at most
100). An update with `held: true` shows only `kind` and `occurred_at`: its
Room is private, or you cannot see it. Ask the requester or a Room member.

`work_updates.attention` is your own unread state: `attention_seq`,
`read_through` and `unread`. Only the requester of a work gets attention,
for its questions and final results. Other members read the same updates
with no unread. Reading does not mark anything read. To mark it, call
`artifactbridge_mark_agent_conversation_read` with the `conversation_id`
and `attention.attention_seq` as `through_attention_seq`. It moves only
your cursor, never backward, and a retry is safe. A value above your
`attention_seq` is refused `invalid_request`: read again.

A receipt `question` with `kind: "human_only"` waits for a person to accept
a proposal in the app. Its `native_handoff_url` opens that proposal. There
is no Room answer, and an agent or coordinator credential cannot accept it.
Ask a member to accept or reject it at that URL. If you cannot see the
proposal, `question` is null. `status` becomes `answered` with the reviewer
and time after the acceptance, or `canceled` after a rejection, a newer
revision or a Stop. An acceptance starts no attempt and no retry. It
sends the work's coordinator one turn to check the outcome.

`work_updates.start_refusals` lists your own starts that were refused
because their Room is private (`room_not_workspace_wide`). Each entry has
`message_id`, `code` and `occurred_at`. Nothing was started. Other members
do not see your refusals. To start the work, choose a Room that the whole
workspace can see and start again with a new accepted message.

## Stop and retry work (member)

`artifactbridge_control_workspace_work` takes one `command` with
`"version": 1` and the `work_id`. Any active member who can see the work's
Room can use `inspect` and `stop`; you do not have to be the requester.
A coordinator credential cannot use this tool.

- `inspect` returns `status`, `state`, `stop_revision` and every attempt of
  every card, each with `delegation_state`, `cancel_outcome` and
  `remote_may_still_run`.
- `stop` needs a `request_id` that you make once and the `stop_revision`
  that you read, as `expected_revision`. The Stop is saved before any task
  is asked to cancel. Then only this work's own live tasks are asked to
  cancel. The same `request_id` returns the same receipt and asks again, so
  repeat it after a lost answer. A changed `stop_revision` refuses
  `revision_conflict`: inspect again and decide.
- `state` after a Stop is `stop_requested`, `stop_unconfirmed` or
  `stopped`. Only `stopped` is a certificate: no new task, start, model call,
  tool call or egress of the work can begin, and every attempt is done
  (refused, never sent, completed, failed, or cancelled with a confirmed
  cancel). `stop_unconfirmed` means that some attempt may still run: its
  task is live, or its cancel was not confirmed (a timeout is not a
  confirmation). Inspect again later to see `stopped`. Completed work stays
  completed, and its results stay: a Stop of completed work returns
  `outcome: "completed_first"` and cancels nothing. A Stop of work that is
  already `stopped` returns its existing certificate. A task of stopped work
  ends failed `work_stopped`.
- A control of work whose Room is no longer visible to the whole workspace
  returns only `{"held": true}` and changes nothing.
- `retry` is for the work's requester only. It needs a `request_id`, a new
  accepted coordinator `message_id` of yours in the work's conversation (one
  retry per message), the `card_id` and the failed `attempt`. It adds the
  next attempt as a separate, visible attempt with its own task. Only the
  latest attempt of a card can be retried, after it failed or was cancelled.

After a Stop, the coordinator's `workspace_card_cleanup` archives the
stopped cards on its board that have not started. Use the ordinary
delegation cancel only for a task that you requested or a Room that you own.

## Refusals and recovery

- `carrier_unavailable`: a coordinator start had no running accepted message
  that matches the request, or a coordinator tried to retry. The member
  starts it with their own credential. Do not relay the start.
- `execution_disabled`: execution is off or not qualified. Ask the operator.
  Do not resend.
- `room_not_workspace_wide`: the Room is private and nothing was started. Pick
  a Room that the whole workspace sees; never broaden a private Room's access,
  and start again with a new accepted member message.
- `factory_disabled`: belongs to the separate legacy factory transport, not
  workspace work. Do not call legacy factory discovery or request tools in this
  flow.
- `card_denied`: the board or agent is not admitted for this work.
- `work_not_admitted`: the work is stopped or finished; no new card.
- `not_found`: no such work, message, conversation or Room for you.
- `retryable_conflict`: another start of the same message is in progress.
  Retry with the same `request_id`.
- `invalid_request`: fix the input; nothing was saved.
- `unsupported_version`: send `"version": 1`; nothing was saved.
- `revision_conflict`: the work's `stop_revision` changed. Inspect again.
- `attempt_not_failed`: retry only the latest attempt, after it failed or
  was cancelled.

## Coordinator cards

The coordinator profile has no native Kanban, cron, delegation or messaging
tools. With the operator's `cards_board` setting it gets confined card
commands for admitted work only: create, inspect, list, link, comment, stop
and retry, on one pinned board, assigned to `remote-worker`. Board, assignee,
profile, model, provider, tenant, path and requester arguments are refused.
Each command is checked and recorded with ArtifactBridge
(`artifactbridge_record_workspace_work_card`) before the board is read or
changed; a refusal leaves the board unchanged. List reads only the work's
recorded cards. A create reuses its idempotency key to
recover. A coordinator retry is refused with `carrier_unavailable`. A card's
text is not authorization; the work record is.

## Coordinator merge

`artifactbridge_merge_workspace_pull_request` asks the gateway merge helper to
merge one pull request of admitted work. Inputs: `work_id`, `pr_number`,
`expected_head_sha` and a `request_id` that you generate once. There is no
repository, base, merge mode or token input; ArtifactBridge takes the
repository and base from the work's execution binding. Only the coordinator of
the work's own conversation can ask. The coordinator action policy lists it
for the plugin only: the plugin's `workspace_merge_pull_request` sends it, and
the model never calls it directly.

The result is a receipt. A new or recoverable request also carries a permit of
at most 60 seconds. Only the helper redeems it; the helper refuses a fork, a
head ref outside `ab/<work_id>/`, another base and a changed head. Read the
receipt `state`: `merged` (with `merge_commit`), `not_merged`, `blocked` (with
`reason`) or `unknown`. A merge never deploys.

- `review_evidence_unavailable`: no accepted specialist review exists for this
  exact pull request and head, or the operator has not qualified protection of
  the repository and base. Do not retry until both exist.
- `merge_disabled`: merge is off for this workspace or its repository
  connection. Ask the operator.
- `work_stopped`: the work is stopped or finished; no merge.
- `foreign_repository` / `foreign_pull_request`: the work names another
  repository, or the pull request is not the target of this work's assigned
  review outcome.
- A receipt `blocked` with `checks_not_passed`: the required checks of this
  exact head have not passed. Wait for the checks, then ask a member as for
  any blocked merge.
- `request_conflict`: the same `request_id` with other inputs, or another
  `request_id` while this pull request has an active merge request. Keep the
  original `request_id` and inputs.
- `carrier_unavailable`: the last merge of this pull request was blocked or
  not merged. A new request needs a new message from a member in this
  conversation; a card update or an operator fix alone is not enough.
- After `possible_write` or `unknown`, send the same request again: you get
  only a read of the same PR, never a second merge. A changed head is never
  retried on the new head. An `unknown` outcome posts one operator handoff in
  the work Room; tell the member an operator must confirm the merge outcome,
  and do not ask for a new merge.

## Coordinator finalize

`artifactbridge_finalize_workspace_work` asks ArtifactBridge to certify
admitted work completed. The coordinator plugin calls it as
`workspace_finalize_work`. Inputs: `version` 1, `work_id`, a `request_id`
that you generate once, the `expected_stop_revision` that you last read and
the `outcome_review_id` of the accepted outcome review. There is no status,
actor, repository or evidence input. Only the coordinator of the work's own
conversation can ask, during a member's message or this work's own card
update. The coordinator action policy lists it for the plugin only; the model
never calls it directly. A member cannot finalize work; a member reads or stops it.

ArtifactBridge certifies only when every fact holds: an accepted review of
every current outcome assignment, the exact retained document version, or
for a pull request the merge at the reviewed head into the approved base and
passed required checks of that head, and no open card effect, merge or human
action. The result is the certificate and the work with `status`
`completed`. A Stop always wins: completed work cannot be stopped, and a
Stop records `completed_first`.

- `outcome_not_ready`: an effect, merge or native human action is still open.
  Nothing was certified; the work stays admitted and can be stopped. Wait for
  it, then send again.
- `outcome_evidence_unavailable`: the review is not an accepted review of
  this work, or its target or evidence does not match. Do not retry until it
  exists.
- `revision_conflict`: the `stop_revision` changed. Read the work again.
- `work_not_admitted`: the work is stopped or already finished.
- `request_conflict`: this work already has a certificate from another
  `request_id` or other inputs. Read the work.
- After a lost answer, send the same request again: you get the same
  certificate.
