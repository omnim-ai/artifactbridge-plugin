# Agent connections

An agent connection lets agents use one GitHub repository through a GitHub
App installation linked to the workspace. The repository's fixed numeric ID
is the key; its owner/name is for display only. A new connection is
disabled, has no operations and has no grants.

## Read (any member, a person or an agent)

When the workspace has agent connections turned on,
`artifactbridge_read_agent_connection` reads one connection:
`connection_id`, and optionally `agent_id` to check that agent's grant. The
result has the provider, the repository (fixed ID and current owner/name),
the allowed operations, the revision, the health (`ready`, `disabled`,
`revoked`, `installation_unavailable`) and `agent_granted`. It never has a
credential, an installation ID or a token. A foreign, missing or hidden
connection or agent answers `connection_not_found` and discloses nothing.

A workspace owner or admin can also list connections with the CLI (below).

## Change (a workspace owner or admin, as a person)

Only an active workspace owner or admin, with their own `artifactbridge
login` sign-in in their own terminal, can authorize, enable, grant or
revoke. An agent credential, an agent-started process and a session started
without a person are refused before any request (`human_required`,
`agent_origin_forbidden`, `wake_session_forbidden`). An agent never
consents to, creates, enables or widens its own access. Hand the step to
the owner or admin:

```
artifactbridge agent connections list [--workspace SELECTOR]
artifactbridge agent connections authorize --installation ID --repository-id ID --name OWNER/NAME
artifactbridge agent connections enable CONNECTION --installation ID --repository-id ID --revision N --operation OP ...
artifactbridge agent connections grant CONNECTION --installation ID --repository-id ID --agent UUID
artifactbridge agent connections revoke CONNECTION --installation ID --repository-id ID [--agent UUID]
```

Operations: `contents_read`, `contents_write`, `pull_requests_read`,
`pull_requests_write`. `enable` replaces the previous set. Without
`--agent`, `revoke` revokes the connection and every grant; it is final.

## Link the GitHub App installation (a workspace owner, attended)

A connection needs a GitHub App installation that is linked to the
workspace. Linking is a person's step in the browser. No MCP tool or CLI
command links an installation, and an agent never handles the GitHub
authorization or its code. When the installation is missing from the
workspace (for example, the install did not return to ArtifactBridge, or
`authorize` cannot find the installation), hand the step to the owner:

1. Give the owner the workspace's Settings address,
   `/?view=settings&section=connections&workspace=<workspace-id>`, and
   the installation ID if you know it.
2. Ask them to use **Link an existing installation**. They authorize on
   GitHub and confirm the workspace in ArtifactBridge.
3. The owner checks that the installation row shows in that workspace.
   Then run `artifactbridge agent connections list` again.

The link checks that the owner's GitHub account has access to the
installation. It does not check GitHub admin rights. A linked installation
gives no agent any access: a connection still needs `authorize`, `enable`
and `grant`. Refusals are fixed: GitHub did not confirm access, the
installation is linked to another workspace, or the authorization must
start again. Do not retry for the owner.

## Failures

- `stale_revision`: the connection changed. Run `list` and use its revision.
- `connection_revoked`: revoke is final. Authorize a new connection.
- `installation_mismatch` / `repository_mismatch`: the IDs do not match the
  connection. Read it again; never guess IDs from the name.
- `workspace_forbidden`: the person is not a workspace owner or admin.
- Unknown outcome (lost reply): run `list` before you try again. Never
  repeat a change blindly.
