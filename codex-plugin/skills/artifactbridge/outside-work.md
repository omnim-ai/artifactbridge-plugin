# Outside work

Use this when work runs in an outside engine (a task board, a CI system, a
scheduler) and the members of an Agent Room must see its state. You report
snapshots; the Room's Work tab shows them. If these tools are not in your tool
list, outside work is not on for this deployment.

## Rules

- Authority comes from your credential. The member behind it owns the sources
  it creates. Source authority is shared by all the owner's supported agents:
  OAuth user, OAuth agent and `afb_` credentials of the same member. No other
  member can report to that source, workspace owners and admins included.
  Hand reporting to the owner; never borrow a credential.
- There is no owner, member or workspace argument. The workspace is the
  credential's.
- Each report is a full snapshot. An omitted optional field is cleared, not
  kept. Send every field you want to keep, every time.
- Revisions come from your engine and only go up. The task revision and each
  run revision are independent. A run report never changes its task.
- Reports are claims, not live health. ArtifactBridge stores what you said
  and when it received it. It never checks the engine.
- Reads do not dispatch work, join Rooms, or contact the engine.
- A closed Room pauses reporting. Keep your backlog. A permitted Room owner can
  reopen the Room; then report the latest snapshot.
- A task stays in the Room where you first reported it.
- Link ArtifactBridge items as typed evidence (`document`, `question`,
  `native_delegation`), never as app URLs. An external URL on the app's own
  origin is refused.

## Report lifecycle

1. Setup, once per engine:
   `artifactbridge_record_outside_source`
   `{"source_key":"ci-main","name":"Main CI"}`. Keep `source_id`. The same
   `source_key` returns the same source with `created: false`. List your own
   sources with `artifactbridge_list_outside_sources` `{"scope":"own"}`.
2. Join the open Room with `artifactbridge_join_agent_room`. Keep your
   `participant_id`.
3. Report the task:
   `artifactbridge_record_outside_task`
   `{"source_id":"<source_id>","outside_task_id":"ENG-42","room_id":"<room_id>","actor_participant_id":"<participant_id>","revision":3,"title":"Ship the importer","source_status":"In Progress","group":"running","responsible_label":"Build bot","outside_url":"https://ci.example.com/eng-42"}`.
4. Report each run (attempt):
   `artifactbridge_record_outside_run`
   `{"task":{"source_id":"<source_id>","outside_task_id":"ENG-42"},"room_id":"<room_id>","outside_run_id":"attempt-1","actor_participant_id":"<participant_id>","revision":1,"source_status":"Passed","group":"done","outcome":"succeeded","started_at":"2026-11-12T09:00:00Z","ended_at":"2026-11-12T09:20:00Z"}`.
   `task` is `{"id":"<task id>"}` or the `{source_id, outside_task_id}` pair,
   never both.
5. When the engine changes, report again with a higher `revision` and the
   whole snapshot.

A write returns a receipt: `disposition`, `record_id`, `revision`,
`received_at` and `app_url`. `applied` saved your snapshot. `duplicate` means
the same revision and snapshot were already saved. `stale` means a higher
revision is saved; `revision` is the saved one.

## Retry after a lost response

A saved revision equal to yours does not prove that your snapshot was saved.
Another agent with the same owner can save other content at that revision.

1. Send the identical report again: the same revision and the same whole
   snapshot.
2. `applied` or `duplicate`: your snapshot is saved.
3. `revision_conflict`: other content is saved at this revision. Read the
   record, reconcile, then report a higher revision.
4. `stale`: a higher revision is saved. Read it; do not send yours again.

Never change the snapshot and keep the revision, and never invent a new run
id for the same attempt. To read one run exactly:
`artifactbridge_read_outside_task`
`{"task":{"source_id":"<source_id>","outside_task_id":"ENG-42"},"outside_run_id":"attempt-1","include_task":false}`.

## Fields

- `group`: `queued`, `running`, `waiting`, `done`, `failed`, `canceled` or
  `unknown`. Map your engine's status to the nearest group; keep its own text
  in `source_status`.
- `outcome` (runs): `succeeded` with `done`; `failed` with `failed`;
  `canceled` with `canceled`; `unknown` or none with `unknown`; none with any
  other group.
- Optional snapshot fields accept `null`. Null and omission both clear the
  field.
- Limits: `source_key` 128 bytes; source `name`, `source_status` and
  `responsible_label` 120 characters; `title` 240; `summary` 2,000; evidence
  `label` 160; ids 256 bytes; URLs `https`, 2,048 bytes; 8 evidence entries;
  8 KiB per normalized snapshot. Times are RFC 3339.
- Evidence entries carry only their kind's keys: `external` `url`, `label`;
  `document` `document_id`, `version_number`; `question` `question_event_id`
  (in the same Room); `native_delegation` `delegation_id`. Readers see only
  the evidence they may read now. Readable native execution shows as a link
  only, with no status or result. Evidence a reader cannot read is removed,
  with its ids, and the record shows `unavailable_evidence: true`.

## Reads

- `artifactbridge_list_outside_tasks`: tasks in Rooms you can read now, newest
  report first. Filter by `source_id`, `room_id` and `groups`. `facet`
  `rooms` or `sources` lists those values for filters.
- `artifactbridge_read_outside_task`: one task with its runs, newest first,
  or exactly one run by `outside_run_id`.
- `artifactbridge_list_outside_sources`: sources of tasks you can read
  (`accessible_work`, the default), or your own sources (`own`).
- Pages use `cursor` (`runs_cursor` for runs). On `page_reset_required`, read
  again from the first page.

## Refusals

A refusal changes nothing.

- `forbidden`: another member owns the source. Hand reporting to that member.
- `not_found`: the Room or task is not readable to you now, or it does not
  exist. Do not retry with other ids.
- `not_participant`: join the open Room and pass your participant id.
- `room_closed`: reporting is paused. Keep the backlog.
- `task_not_found`: report the task before its runs.
- `identity_conflict`: the record exists in another Room. Read and reconcile.
- `revision_conflict`: this revision holds another snapshot. Read, reconcile,
  then report a higher revision.
- `invalid_report`: fix the shape, bounds, group, outcome, time or URL.
- `retryable_conflict`: send the same report again after a short wait.
