# Acceptable Use Policy

This policy defines the boundaries for using AI tools within the Inverter Network. It applies to all team members and contributors who use AI assistants in their work.

## Table of Contents

1. [General Principles](#1-general-principles)
2. [Permitted Uses](#2-permitted-uses)
3. [Restricted Uses](#3-restricted-uses)
4. [Prohibited Uses](#4-prohibited-uses)
5. [Data Handling](#5-data-handling)
6. [Accountability](#6-accountability)
7. [Review & Updates](#7-review--updates)

---

## 1. General Principles

- **AI is a tool, not a replacement.** All AI-generated output must be reviewed and validated by a human before being accepted.
- **You own what you ship.** The person who submits AI-assisted work is responsible for its correctness, security, and quality — not the AI.
- **Transparency matters.** When AI has played a significant role in producing an artifact (e.g. an architecture proposal, a security analysis), this should be noted.

## 2. Permitted Uses

The following uses are encouraged and approved:

- **Code generation & completion** — Using AI to draft code, boilerplate, tests, or configuration files, provided the output is reviewed.
- **Code review assistance** — Having AI flag potential issues, suggest improvements, or explain unfamiliar code.
- **Documentation** — Drafting, improving, or translating technical documentation.
- **Debugging & analysis** — Using AI to help diagnose bugs, explain error messages, or reason about complex logic.
- **Learning & exploration** — Asking AI to explain concepts, compare approaches, or prototype ideas.
- **Commit messages & PR descriptions** — Generating clear, well-structured commit messages and PR summaries.

## 3. Restricted Uses

The following uses require caution and may need additional review:

- **Security-sensitive code** — AI-generated code that handles authentication, authorization, cryptography, or financial logic must undergo thorough manual review and, where applicable, formal audit.
- **Smart contract logic** — Any AI-generated Solidity or on-chain logic must be treated as a first draft. It must pass the full review and testing pipeline defined in the [Code Standard](../../code/README.md) and [Security Standard](../../security/README.md).
- **External communications** — AI-drafted content intended for public release (blog posts, announcements, governance proposals) must be reviewed for tone, accuracy, and alignment with Inverter's voice.

## 4. Prohibited Uses

The following uses are not allowed:

- **Submitting unreviewed AI output** — Never merge, publish, or deploy AI-generated work without human review.
- **Sharing sensitive data with AI tools** — Do not paste private keys, secrets, credentials, personal data, or confidential business information into any AI tool, unless the tool is self-hosted or contractually guaranteed to not retain data.
- **Misrepresenting AI work as human-only** — If asked whether AI was used, always answer honestly.
- **Bypassing review processes** — AI does not replace code review, security audits, or any other established quality gate.

## 5. Data Handling

- **Assume public exposure.** Treat any input to a cloud-hosted AI tool as potentially accessible beyond your session, unless the provider's data policy explicitly states otherwise.
- **Prefer tools with data retention opt-outs.** Where available, disable training on your data (e.g. Claude's and ChatGPT's enterprise/API tiers).
- **Never input secrets.** Private keys, API tokens, passwords, and environment files must never be pasted into AI tools.
- **Scrub before pasting.** When sharing code snippets for analysis, remove or redact any hardcoded credentials, internal URLs, or personally identifiable information.

## 6. Accountability

- **Authorship** — The contributor who submits AI-assisted work is the author of record and bears full responsibility.
- **Review** — AI-generated code is held to the same review standards as human-written code. Reviewers should not lower their bar because AI was involved.
- **Incidents** — If a bug or vulnerability is traced to AI-generated code, the post-mortem follows the same process as any other incident. The use of AI should be noted as a contributing factor where relevant.
