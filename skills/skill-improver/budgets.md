# Skill-improver budgets

This skill spends the human's own tool-call quota. An attended companion turn
allows **16 tool calls**, so the loop must fit inside that ceiling with room to
spare. A claimed external run and a local agent have more headroom, but they
use the same budget: the caps below are what keeps the loop reluctant, not what
the runtime happens to allow.

## The attended call table

| Call | Count |
| --- | --- |
| `artifactbridge_list_skills` | 1 (catalog lookup, only when you must find this skill) |
| `artifactbridge_read_skill` for `skill-improver` | 1 (loading this skill) |
| `artifactbridge_score_skill_evidence` | **1. Exactly one per run.** |
| `artifactbridge_read_room_events` | **0.** Use `targets[].cited_events`. Never re-fetch. |
| Per `existing_skill` target | 1 `read_skill` + 1 `read_document` (`include_atoms: true`) + 1 `propose_document_patch` |
| Per `new_skill` target | 1 `create_document` |
| Report | 1 `create_document` |
| **Total ceiling** | **16.** Three existing-skill targets cost 3 + 9 + 1 = 13, plus the catalog and skill load. Stop there. |

The score tool is itself rate limited: **3 calls per 60 seconds per API token.**
One call per run stays far under it. If you get `rate_limited`, wait for
`retryAfterSeconds` and try once more. Never loop on it.

## The hard caps

- **One score call per run.** The scorecard is the run's only evidence. A
  second call rescores the same rooms and cannot produce new targets.
- **Zero room-event reads.** `targets[].cited_events` carries at most 5 events
  per target, each with an excerpt of at most 200 characters. That is the
  evidence. `artifactbridge_read_room_events` adds nothing and costs the turn.
- **At most three patches per run.** `targets` holds at most three, already
  ranked worst first. Never propose a fourth from `failed_rooms`, from
  `rooms`, or from memory. The server does not enforce this cap — you do.
- **One report per run.** One `create_document` with `review_mode: "working"`.
  Do not refresh it more than once in the same turn.
- **Zero accept or reject calls.** Not one, ever.

## Server bounds the scorecard already applies

You do not need to re-derive these; read them from `window` and report them.

- **Lookback: 30 days by default**, at most 45.
- **Sample: 12 rooms by default**, at most 20.
- **Joined discovery: 50 rooms.** The sample is drawn from the rooms you own
  plus the rooms you joined, and the joined half is capped at 50. Say so in the
  report — it is the sample's known bias.
- **Events per room: the newest 500.** A busier room is scored on its newest
  500 typed events, the same cap the ambient room digest uses.
- **Skills considered: active managed-document skills only.** Bundled skills
  are excluded, so a bundled name can never dominate the fit score.

## What to do when a budget runs out

- **Out of tool calls before the report:** stop proposing. Write the report
  with the targets you did handle and name the ones you did not.
- **Rate limited on the score call:** wait `retryAfterSeconds`, retry once, then
  tell the human the run could not start.
- **A propose call conflicts:** the error names `current_document_version_id`.
  Read head again and rebase once. A second conflict means someone is editing
  the document now — skip that target and say so.
- **A private create is rejected:** record `private_writes_disabled` in the chat
  reply or on `artifactbridge_workflow_finish_run` and stop. Never retry as
  workspace-visible.
- Every case ends the same way: report what happened. A partial run is a valid
  run. It never fails silently.

## Why these numbers

- The 16-call ceiling is the attended companion turn's hard limit. The table
  above is what fits inside it with three targets.
- Zero event re-reads is the reason the score tool returns excerpts at all. One
  room's event page can hold 100 events; twelve rooms cannot fit in one turn.
- Three patches is a reluctance rule, not a throughput rule. A human reviewing
  more than three skill edits at once is not reviewing them.
- One report keeps the run's receipt in one place. A scorecard is an
  operational record, not a published procedure, which is why it is a
  **working** document and the skill edits are governed proposals.
