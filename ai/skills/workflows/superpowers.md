# Superpowers

[Superpowers](https://github.com/obra/superpowers) by Jesse Vincent — an agentic skills framework that ships installers for Claude Code, Codex (CLI + App), Cursor, OpenCode, GitHub Copilot, and Gemini (MIT license). It's the first skill listed in the Inverter recommended bundle.

## Why it's in the Inverter bundle

Inverter dev work involves smart contracts, security-sensitive logic, and long-form features that benefit from up-front design and rigorous testing. Superpowers enforces a workflow that matches how we want AI-assisted development to happen:

1. **Brainstorm before code** — Socratic refinement, design doc, user sign-off
2. **Plan in small tasks** — exact file paths, expected diffs, verification steps
3. **TDD by default** — red → green → refactor; no implementation without a failing test first
4. **Review before merge** — checks against the plan, severity-ranked issues
5. **Clean finish** — tests pass, branch decision presented, worktree cleaned up

The skills auto-trigger; the agent picks the right phase on its own. No manual invocation needed.

## When to use

- Non-trivial features, bugfixes, or refactors
- Anywhere consistent test-driven, plan-driven output across the team matters
- Long autonomous runs (the agent stays on-plan for an hour+ without deviating)

## When to skip

- One-line fixes or trivial edits where the workflow ceremony exceeds the value
- Throwaway exploratory work

## Install

For Claude Code (Inverter marketplace — recommended):

```
/plugin marketplace add InverterNetwork/standard
/plugin install superpowers@inverter-standard
```

For Codex, Cursor, OpenCode, Copilot, or Gemini, see the per-agent install table in [`AGENTS.md`](../../../AGENTS.md). Restart your agent after installing; verify the skills are loaded (e.g. `/plugin` in Claude Code).

## What's inside

The plugin ships ~14 skills. The core workflow:

| Phase | Skill |
|-------|-------|
| 1 | `brainstorming` |
| 2 | `using-git-worktrees` |
| 3 | `writing-plans` |
| 4 | `subagent-driven-development` / `executing-plans` |
| 5 | `test-driven-development` |
| 6 | `requesting-code-review` |
| 7 | `finishing-a-development-branch` |

Plus debugging (`systematic-debugging`, `verification-before-completion`), collaboration (`dispatching-parallel-agents`, `receiving-code-review`), and meta (`writing-skills`, `using-superpowers`) skills.

## Combine with project skills

Superpowers handles the *how* of working. Pair it with Inverter's domain skills for the *what*:

- [Solidity Testing](../solidity-development/testing/README.md) — building complete test suites
- [QA Testing](../qa-testing/README.md) — guided product testing

## Pin and audit

The Inverter marketplace currently lists Superpowers at version `5.0.7` ([source repo](https://github.com/obra/superpowers)). When bumping the pinned version in [`.claude-plugin/marketplace.json`](../../../.claude-plugin/marketplace.json), apply the same audit discipline as for submodules — see [Importing Submodules](../contributing/adding-content.md#importing-submodules). Diff upstream `hooks/` and `scripts/` for anything that runs at install time.

## References

- Source: [obra/superpowers](https://github.com/obra/superpowers) (MIT)
- Release announcement: [blog.fsck.com/2025/10/09/superpowers/](https://blog.fsck.com/2025/10/09/superpowers/)
- Inverter marketplace: [`.claude-plugin/marketplace.json`](../../../.claude-plugin/marketplace.json)
