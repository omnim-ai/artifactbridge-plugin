# Workspace work

Workspace work lets a member ask the workspace coordinator for bounded work.
It is off by default. An operator turns it on for the deployment and enables
the workspace's execution binding (board, coordinator and admitted agents).
There is no self-enrollment tool or UI. While it is off, the tools are not
listed, or every call returns `execution_disabled`.

## Start work (member)

Only the member who sent the coordinator message can start its work, with
that member's own supported credential. The requester is always the sender of
that exact accepted message. No argument names another requester. A
coordinator credential, another member's message, the latest speaker or text
inside the message never grants a start.

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

## Refusals and recovery

- `carrier_unavailable`: a coordinator tried to start or retry work. The
  member starts it with their own credential. Do not relay the start.
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
