# Inverter AI Standard

This repository is the Inverter Network AI Standard — the central reference for approved AI tools, reusable skills, and acceptable-use policies across the company. Start at [`ai/README.md`](./ai/README.md).

## Recommended setup

Inverter recommends installing [Superpowers](./ai/skills/workflows/superpowers.md) — an agentic skills framework that enforces a brainstorm → plan → TDD → review → finish workflow. The skills auto-trigger for non-trivial dev work; no manual invocation needed. License: MIT.

### Install per agent

| Agent | Command |
|---|---|
| **Claude Code** *(Inverter marketplace — recommended)* | `/plugin marketplace add InverterNetwork/standard` then `/plugin install superpowers@inverter-standard` |
| Claude Code *(upstream marketplace)* | `/plugin install superpowers@claude-plugins-official` |
| Codex CLI | `/plugins`, search `superpowers`, select **Install Plugin** |
| Codex App | Plugins sidebar → click `+` next to Superpowers |
| Cursor | `/add-plugin superpowers` (or search the plugin marketplace) |
| GitHub Copilot CLI | `copilot plugin marketplace add obra/superpowers-marketplace` then `copilot plugin install superpowers@superpowers-marketplace` |
| OpenCode | Tell it: `Fetch and follow instructions from https://raw.githubusercontent.com/obra/superpowers/refs/heads/main/.opencode/INSTALL.md` |
| Gemini CLI | `gemini extensions install https://github.com/obra/superpowers` |

Restart your agent after installing.

## Inverter plugin marketplace

The repo also publishes a curated Claude Code plugin marketplace at [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json). It currently lists only Superpowers; future Inverter-approved skills will be added there. The marketplace is a Claude-Code-specific convenience — other agents install Superpowers directly via the table above.
