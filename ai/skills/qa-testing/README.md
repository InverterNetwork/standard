# QA Testing

Skills for AI-guided quality assurance — structured prompts that turn any AI chat tool into a QA companion that walks testers through a product, asks the right questions, and writes up findings as GitHub issues.

## How It Works

The base skill ([Guided User Testing](./guided-user-testing.md)) provides a project-agnostic testing methodology with three reporting flows:

- **Userflow Journal** — guided walkthrough of a lifecycle phase, capturing expectations before and reality after
- **Friction Point** — quick 2-minute capture of a UX moment that felt off
- **Bug Report** — structured report for something broken

The methodology is designed to be adapted for specific products and environments. Fill in the placeholder sections (phases, glossary, feature areas, templates) to create a ready-to-use prompt for your project.

## Skills

| Skill | Description |
|-------|-------------|
| [Guided User Testing](./guided-user-testing.md) | Base prompt template — project-agnostic testing agent with configurable phases, roles, and issue templates |
| [Floor Markets Phase 2](./floor-markets-phase-2.md) | Ready-to-use extension for Floor Markets Phase 2 testnet (Prelaunch → Whitelist → Public → Post Presale → Live Trading) |

## Extending for a New Project

1. Start from [Guided User Testing](./guided-user-testing.md)
2. Replace all `[PLACEHOLDER]` sections with your project's details — see the Customisation Points table in that file for what to fill in
3. Add your phase guides following the Orient → Plan → Execute → Reflect → Probe → Compile pattern
4. Inline your GitHub issue templates in the appendix
5. Save the result as a new file in this folder (e.g. `my-project-v1.md`)
