# Approval rules for imports

A human accepts an import plan. The server applies only the accepted plan. These
rules protect that boundary. Follow every rule.

## Plan and apply are separate steps

- Create the plan with `artifactbridge_plan_document_import`. This step does not
  change documents.
- Apply the plan with `artifactbridge_apply_document_import_plan`. This step is
  a separate tool call.
- Never call apply in the same step that first shows the plan to the human.
- This rule governs agent tool calls. In the web Inbox, AI-2568 performs the
  accept and the apply in one human action when the human presses Import on
  the proposal's Inbox item; that one press IS the explicit human decision.

## Only a human accepts

- `artifactbridge_accept_document_import_plan` is an OAuth-human-only decision.
- The server refuses an accept call that carries an `afb_` agent token.
- Never accept a plan yourself.
- Silence is not acceptance. A prior general instruction is not acceptance.
  Plan generation is not acceptance.
- Wait for one explicit human decision.

## Server acceptance is the source of truth

- A chat message that says "yes" is not server acceptance.
- Confirm the plan `status` is `accepted` on the server before you apply.
- The `accept` step binds the exact manifest digest. Apply uses the same plan
  id.

## A changed plan needs a new acceptance

- The human accepts one exact manifest digest.
- When the plan changes, the digest changes. Show the new digest. Wait for a
  new acceptance.
- The `accept` call sends `expected_revision` and `expected_digest`. The server
  rejects a stale digest as `plan_changed`. Create and show a new plan, then
  ask for a new acceptance.

## An expired plan needs a new plan

- Each plan has an expiry time.
- When the plan expires, create a new plan. Show it. Wait for a new acceptance.

## Governed documents keep their proposal gate

- A changed governed document becomes a proposal, not a direct write.
- Report each `review_request_id`.
- Do not accept the proposals for the human. This skill stays pinned to the
  accepted versions.
- Do not bypass the document proposal gate or the skill re-pin rules.
