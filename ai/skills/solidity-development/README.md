# Solidity Development Skills

AI skills for Solidity smart contract development at Inverter. Each subfolder groups skills by activity.

## General

Standalone skills that apply across the development lifecycle.

| Skill | What it does |
|-------|--------------|
| [commit-messages.md](./commit-messages.md) | Generate commit messages from staged changes using the Inverter format |

## Testing

A structured, step-by-step workflow for building a complete Foundry test suite for a Solidity contract. The skills are designed to be used in order — each step builds on the output of the previous one.

→ **[Testing Skills](./testing/README.md)**

| Step | Skill | What it does |
|------|-------|--------------|
| 1 | [test-skeleton.md](./testing/test-skeleton.md) | Scaffold the test file with sections, placeholders, and exposed mock |
| 2 | [test-modifiers.md](./testing/test-modifiers.md) | Isolation tests + in-position checks for all modifiers |
| 3 | [test-internal-functions.md](./testing/test-internal-functions.md) | Business logic tests for internal functions via exposed mock |
| 4 | [test-external-functions.md](./testing/test-external-functions.md) | Integration tests for the public API |
| · | [test-fuzz.md](./testing/test-fuzz.md) | Fuzz testing methodology — used alongside Steps 3 and 4 |
| 5 | [test-e2e.md](./testing/test-e2e.md) | End-to-end tests across multiple modules |
| 6 | [test-invariant.md](./testing/test-invariant.md) | Invariant tests with handlers and ghost variables |
