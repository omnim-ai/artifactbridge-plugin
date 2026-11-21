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

The coordinator action policy does not yet offer the start to the
coordinator. When it does, the coordinator can start work only for the
accepted message of the turn it is running. Hermes reports that message to
the coordinator plugin; it is never a tool argument. The requester is that
message's sender, not the coordinator's credential owner. Without a supported
Hermes, or for a message whose turn has ended, the start returns
`carrier_unavailable`.

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
  completed, and its results stay. A task of stopped work ends failed
  `work_stopped`.
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
- `room_not_workspace_wide`: the Room is private. Pick a Room that the whole
  workspace sees; a private Room is never broadened.
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
the work's own conversation can ask. The tool is not in the coordinator action
policy yet, so a coordinator cannot reach it until the operator qualifies a
policy that lists it.

The result is a receipt. A new or recoverable request also carries a permit of
at most 60 seconds. Only the helper redeems it; the helper refuses a fork, a
head ref outside `ab/<work_id>/`, another base and a changed head. Read the
receipt `state`: `merged` (with `merge_commit`), `not_merged`, `blocked` (with
`reason`) or `unknown`. A merge never deploys.

- `review_evidence_unavailable`: no specialist review and test evidence exists
  for this exact head. Do not retry until it exists.
- `merge_disabled`: merge is off for this workspace or its repository
  connection. Ask the operator.
- `work_stopped`: the work is stopped or finished; no merge.
- `foreign_repository` / `foreign_pull_request`: the work names another
  repository, or the pull request is not this work's.
- `request_conflict`: the same `request_id` with other inputs. Keep the
  original inputs.
- After `possible_write` or `unknown`, send the same request again: you get
  only a read of the same PR, never a second merge. A changed head is never
  retried on the new head.
