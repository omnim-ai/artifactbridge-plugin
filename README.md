# ArtifactBridge plugins

Official ArtifactBridge plugins for Claude Code, Codex, and clients that support
the Agent Plugins 1.0.0 standard. ArtifactBridge is a
governed document layer for AI agents: agents read, comment, ask, and propose;
humans approve changes. Learn more at
[artifactbridge.com](https://artifactbridge.com) and
[artifactbridge.com/docs](https://artifactbridge.com/docs).

Installing a plugin wires the ArtifactBridge MCP connection (browser OAuth, no
token paste) and loads the `artifactbridge` skill pack.

## Install

### Cursor and Grok Bot submission

The repository root is an Agent Plugin: `plugin.json`, `mcp.json`, and `skills/`.
Cursor supports this portable format. Marketplace review and Grok Bot catalog
availability are pending; this repository does not claim an approved listing.

For local Cursor testing, copy this repository into
`~/.cursor/plugins/local/artifactbridge`, reload Cursor, and confirm that
ArtifactBridge appears in Customize. Complete browser OAuth when prompted.
Local imports must be allowed by your team policy. An existing marketplace
installation with the same name takes precedence over the local package.

Once listed in Grok Bot, open Plugins, find ArtifactBridge, add it, and complete
the browser authorization. A listing and a successful Grok Bot connection must
be verified before treating this path as available.

Try: "Find the team's onboarding documents and summarize the current process."
Then: "Propose a clarification to this managed document for human review."
Review and accept proposals in ArtifactBridge. The plugin does not grant agents
authority to approve external source writes.

The MCP server uses Streamable HTTP at `https://app.artifactbridge.com/mcp`.
Authentication is client-managed OAuth; no API key or local server is required.
An ArtifactBridge account is required. If authentication fails, reconnect in
the client's plugin settings. Do not paste credentials into chat.

See [Cursor's plugin documentation](https://cursor.com/docs/plugins) and
[Grok Bot's connection guide](https://cursor.com/help/grok-bot/connect-plugins).

### Claude Code and Codex

**Claude Code:**

```sh
claude plugin marketplace add omnim-ai/artifactbridge-plugin
claude plugin install artifactbridge
```

**Codex** (reads the same marketplace manifest):

```sh
codex plugin marketplace add omnim-ai/artifactbridge-plugin
codex plugin install artifactbridge-codex
```

After install, run `/mcp` in your session to complete the browser OAuth sign-in.
An ArtifactBridge account is required.

## Contents

- `plugin.json`, `mcp.json`, `skills/` — the portable Agent Plugin, with the
  same skills as the Claude Code package and no local runtime dependency.
- `logo.png` — ArtifactBridge's square icon for the marketplace application.

- `claude-code/` — the Claude Code plugin: MCP connection plus the
  `artifactbridge`, `import-repo`, `import-obsidian`, and `crawl-and-propose`
  skills.
- `codex-plugin/` — the Codex plugin: the same skills with Codex MCP wiring.
- `.claude-plugin/marketplace.json` — the marketplace manifest both runtimes
  read.

## About this repository

This repository is a read-only publish target, mirrored automatically from the
ArtifactBridge monorepo. Do not open pull requests here — they cannot be
merged. Report issues and requests at
[artifactbridge.com/docs](https://artifactbridge.com/docs) or to your
ArtifactBridge workspace contact.

## Privacy and terms

- Privacy policy: <https://artifactbridge.com/privacy>
- Terms: <https://artifactbridge.com/terms>
