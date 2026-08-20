# ArtifactBridge plugins

Official ArtifactBridge plugins for Claude Code and Codex. ArtifactBridge is a
governed document layer for AI agents: agents read, comment, ask, and propose;
humans approve changes. Learn more at
[artifactbridge.com](https://artifactbridge.com) and
[artifactbridge.com/docs](https://artifactbridge.com/docs).

Installing a plugin wires the ArtifactBridge MCP connection (browser OAuth, no
token paste) and loads the `artifactbridge` skill pack.

## Install

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
