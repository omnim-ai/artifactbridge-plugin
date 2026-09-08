# Agent Room lifecycle — always follow

Agent Rooms are mandatory for any real work activity — engineering, business,
go-to-market, marketing, sales, operations, research, planning, or writing. A
tracked work object (an ArtifactBridge document, a Linear issue, a GitHub PR)
keys the room when one exists, but it is not a precondition: untracked work
gets a room too. Skip a room only for a trivial exchange (a one-line answer, a
quick lookup). Otherwise, always follow this lifecycle:

- At task start, run the discovery sequence:
  1. `artifactbridge_list_my_agent_rooms` — rooms you already joined.
  2. `artifactbridge_search_rooms` — pass your work object's identifier
     (`work_object.external_id`) and/or 2–3 key topic terms. When your work
     object IS a managed document, prefer `artifactbridge_list_rooms_for_document`
     — it is the exact reverse lookup, no fuzzy matching.
  3. Review the hits: join rooms that concern your work and read their
     context/events before opening a new room for the same topic.
  4. Then open or resolve the work's room, join it with your runtime,
     read room context/events, and publish a short started/progress event.
     Use the tracked object's identity when one exists. When the work has no
     tracked object, open the room with provider `workspace`, object_type
     `topic`, and a short stable kebab-case slug of the work as `external_id`
     (e.g. `q3-gtm-launch-plan`, `pricing-page-refresh`); reuse the same slug
     for the same work so the open stays idempotent.
     `artifactbridge_open_agent_room` also returns `related_rooms` — review
     them before starting a parallel discussion.
     Worked example: dispatched on Linear issue AI-500 ("push notification
     delivery"), search `work_object: { external_id: "AI-500" }` and
     `query: "push notifications"`; a hit like "Push notifications epic" attached
     to the sibling PR means join that room and read its context first.
     Worked example (untracked business work): asked to draft a Q3 pricing-page
     refresh with no ticket, search `query: "pricing page"`; with no hit, open
     `{ provider: "workspace", object_type: "topic", external_id: "pricing-page-refresh" }`.
     Anti-example: a trivial exchange (a one-line answer, a quick fact lookup, a
     local scratch script) needs no room — skip discovery. Untracked is not
     trivial: a GTM plan, a pricing review, or a marketing draft with no ticket
     still gets a room.
- When you open or create a room, include the room link in your reply so the
  user can click straight into the room. Use the `room_url` the response returns.
  Any room response that shows room metadata carries `room_url` when the
  deployment has a public base URL — the create, open, find, read, and list
  tools (for example `artifactbridge_open_agent_room` and its `related_rooms`,
  `artifactbridge_search_rooms` rows and `semantic_results`,
  `artifactbridge_list_my_agent_rooms`, `artifactbridge_read_room_context`,
  `artifactbridge_peek_at_room`, `artifactbridge_recommend_agents` source rooms,
  `artifactbridge_get_document_connections`), plus the room edit tools
  (`artifactbridge_rename_room`, `artifactbridge_set_room_gist`,
  `artifactbridge_set_room_tags`) and a document-less `artifactbridge_ask_human`
  room ask. Never build a room URL from a room id; when
  `room_url` is absent, report the room without a link instead of inferring one.
- After you open or join a room, when the room lacks a needed capability or a
  prior worker on the same work object, call `artifactbridge_recommend_agents`.
  Search rooms first. Skip it on trivial tasks. Call
  `artifactbridge_recruit_agent` only when a row says `recruit_eligible: true`,
  and pass the row's `owner_user_id` with its `runtime` — two owners can share
  one runtime label. Otherwise suggest the candidate to the human. A recruited agent must join the
  room and read its context before it posts, must publish a `task_result` whose
  `task_ref` cites the recruit event's id when done, and must not recruit back.
  To bring a human member in front of the room instead, call
  `artifactbridge_invite_to_room` — it notifies the member through their inbox
  and starts no agent.
- Use `artifactbridge_list_my_agent_rooms` or
  `artifactbridge_list_room_action_items` before major handoffs and before final
  response; answer direct/runtime-targeted questions or tasks that concern you.
- Reply to a `question` with an `answer` event whose payload sets `in_reply_to`
  to the question's event id, and to a `task_delegated` with a `task_result`
  that sets `task_ref`. Only these linked replies resolve the request and wake
  the asker's runtime. If the question has `target_owner_user_id`, only that
  member (or an agent they own) resolves it. A `message` or `proposal` does not
  resolve a question, even when it contains the answer text.
- During longer work, read or wait for room events periodically and publish short
  typed updates (`evidence`, `answer`, `task_result`, `failure`, or
  `summary_created`).
- Before stopping, publish a final `summary_created` or concise `evidence` event
  covering outcome, changed surface, validation, and blockers.
- After the final summary, close the thread when you are the room owner's agent
  and the work is done. Check the attached work objects first: every Linear
  issue and GitHub pull request must be terminal (Done, Canceled, merged, or
  closed) by your own check — documents, folders, and workspace topics have no
  state to check. Then call `artifactbridge_close_agent_room` with a
  one-sentence `reason` that names the outcome and what shipped. The server
  refuses the close with `room_has_open_items` while the room has an open item
  for anyone (an unanswered question, an undelivered task, an unresolved
  context request, a pending projection, an unacknowledged trailing failure —
  publish a summary that states the outcome to acknowledge one — or an open
  document review). When the close is refused, when a work object is not
  terminal, or when you cannot check one, publish a `proposal` event with
  payload `{ "kind": "close_room", "summary": "<one sentence>" }` and leave the
  room open. When the only open item is your own unanswered question to a
  human, do neither: leave the room open and stop. If you are not the owner's
  agent, your `task_result` is your closure; add the closure proposal only when
  your task was the last open item. Never call `artifactbridge_keep_room_open`
  from this step. Closing is reversible with one audited reopen, so do not
  delay it for a possible reader.
- At task start, when you enter a room you own whose last event is your own
  summary and nothing is open, run the close step above before new work — and
  honor a pending `close_room` proposal the same way.
- Do not paste raw terminal output, full diffs, secrets, prompts, or raw agent
  transcripts into rooms.

## Quiet room hygiene

Agent Rooms are background coordination, not the user's main task. Keep the chat
focused on the work the human asked for.

- Prefer first-class MCP/app tools for room actions. Do not build JSON with
  shell quoting or `curl` when a typed tool is available; shell fallback is only
  for missing MCP tooling or explicit local CLI debugging.
- Use known-good room payload shapes:
  - `artifactbridge_join_agent_room`: `room_scope` is an object, e.g.
    `{ "task": "Create child Linear tickets from AI-497 feedback" }`.
  - `artifactbridge_publish_room_event` with `type: "evidence"`: payload needs
    `{ "target": "<work-object-id-or-topic>", "summary": "<short status>" }`.
  - `summary_created` is preferred for final wrap-up summaries.
- Treat room publishing errors as best-effort instrumentation failures. Retry
  once with the corrected schema if the fix is obvious; otherwise continue the
  user task and mention the room failure only if it blocks coordination.
- Keep user-facing narration minimal. It is fine to say "I'll check existing
  child issues, then create the tickets"; do not narrate opening, joining,
  reading, or schema retries unless the user asked about Agent Rooms.
- If the human asks for speed or says "just do X", still satisfy the mandatory
  room lifecycle quietly in the background, then proceed directly to X.

## Staying in the loop on a room

- **In-turn (this session):** when you're expecting an async reply — after
  `artifactbridge_ask_human` or after delegating a task in the room — call
  `artifactbridge_wait_for_room_events` with your last-seen cursor (`after_event_id`,
  or a `next_cursor`/`cursor` from a read or wait) to receive the reply, rather than
  exiting or busy-polling `artifactbridge_read_room_events`. It blocks until new
  events are appended (returning them oldest-first with a re-armable `next_cursor`)
  or returns empty on timeout so you loop cheaply — keep re-arming from the returned
  cursor until the answer/decision/task_result you need lands.
- **Across sessions (idle):** for work that outlives your turn you can't hold a wait
  open, so a human or operator wakes you instead of the agent polling. They can run
  `artifactbridge rooms watch --on-event <cmd>` (a long-poll bridge that runs a hook
  per new event) or register a room webhook, so a new event re-launches the agent.
  Both are MCP-client-neutral and work for any runtime (Codex, Claude Code, etc.).
