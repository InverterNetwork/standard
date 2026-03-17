# AI Skills

Skills are reusable, documented patterns for getting reliable results from AI tools. Each skill targets a specific task — like reviewing code, writing commit messages, or debugging — and provides structured prompts, context guidelines, and examples.

## Why Skills?

- **Consistency** — Everyone on the team gets similar quality output for the same type of task.
- **Efficiency** — No need to craft prompts from scratch every time.
- **Knowledge sharing** — Effective techniques are captured once and reused by all.

## Skill Categories

### Solidity Development

Skills specific to Solidity smart contract development.

| Area | Skills | Guide |
|------|--------|-------|
| [Testing](./solidity-development/testing/README.md) | Skeleton · Modifiers · Internal Functions · External Functions · Fuzz · E2E · Invariant | Step-by-step workflow for building a complete test suite |
| General | [Commit Messages](./solidity-development/commit-messages.md) | Generate commit messages from staged git changes |

→ See the [Solidity Development overview](./solidity-development/README.md) for details.

### Contributing

How to add new content to the AI standard.

| Skill | Description |
|-------|-------------|
| [Adding Content](./contributing/adding-content.md) | How to add tools, skills, categories, or sections to `ai/` |

## Writing a New Skill

When documenting a new skill:

1. Create a markdown file in the appropriate category folder
2. Use the following structure:
   - **Title & purpose** — What the skill does and when to use it
   - **Prerequisites** — Required context, tools, or setup
   - **Prompt template** — The core prompt(s) to use
   - **Tips** — What works well, what to watch out for
   - **Examples** — Before/after or input/output samples
3. Add the skill to the table in this README and in the main [AI README](../README.md)

For the full guide on structure, naming, and the "walk up the tree" rule, see [Adding Content](./contributing/adding-content.md).
