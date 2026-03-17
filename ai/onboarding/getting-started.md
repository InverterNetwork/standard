# Getting Started with AI at Inverter

Welcome! This guide will help you set up and start using AI tools as part of your development workflow at Inverter.

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Setting Up Your Tools](#2-setting-up-your-tools)
3. [Your First Tasks](#3-your-first-tasks)
4. [Key Rules to Remember](#4-key-rules-to-remember)
5. [Where to Go Next](#5-where-to-go-next)

---

## 1. Prerequisites

Before you begin, make sure you have:

- [ ] Access to the Inverter GitHub organisation
- [ ] Read the [Acceptable Use Policy](../rules/acceptable-use.md)
- [ ] An account for at least one approved AI tool (see [Tools](../tools/README.md))

## 2. Setting Up Your Tools

### Cursor (Recommended for Development)

1. Download [Cursor](https://cursor.com) and install it
2. Sign in with your account
3. Open your Inverter project repository
4. Familiarise yourself with the key features:
   - **Tab completion** — AI-powered autocomplete as you type
   - **Chat** (`Cmd/Ctrl + L`) — Ask questions about your codebase
   - **Agent** (`Cmd/Ctrl + I`) — Multi-step code generation and editing
5. See the [Cursor guide](../tools/cursor.md) for Inverter-specific configuration

### Claude

1. Access Claude at [claude.ai](https://claude.ai) or via API
2. Use Claude for longer analysis, document review, or complex reasoning tasks
3. See the [Claude guide](../tools/claude.md) for prompt tips and best practices

### ChatGPT

1. Access ChatGPT at [chat.openai.com](https://chat.openai.com) or via API
2. Use ChatGPT for brainstorming, quick questions, and general drafting
3. See the [ChatGPT guide](../tools/chatgpt.md) for configuration details

## 3. Your First Tasks

Try these tasks to get comfortable with AI-assisted development:

### Task 1: Explain Existing Code

Pick a file from the codebase you're unfamiliar with and ask an AI tool to explain it:

> "Explain what this contract does, focusing on the public interface and any security considerations."

### Task 2: Write a Test

Take an untested or under-tested function and ask AI to help generate test cases:

> "Write unit tests for this function. Cover happy path, edge cases, and error conditions."

### Task 3: Improve a Commit Message

After staging your changes, ask AI to draft a commit message:

> "Given this diff, write a concise commit message following conventional commits format."

### Task 4: Review Your Own Code

Before submitting a PR, paste your diff and ask for a review:

> "Review this diff for bugs, security issues, and style improvements."

## 4. Key Rules to Remember

1. **Always review AI output** — Never merge or deploy without reading and understanding every line.
2. **Never share secrets** — No private keys, API tokens, or credentials in AI tools.
3. **You are responsible** — AI-generated code is your code once you submit it.
4. **Be transparent** — Note when AI played a significant role in producing work.
5. **Follow existing standards** — AI output must meet the same quality bar as human-written code (see [Code Standard](../../code/README.md)).

## 5. Where to Go Next

| Resource | Description |
|----------|-------------|
| [Tools Overview](../tools/README.md) | Detailed guides for each approved tool |
| [Skills](../skills/README.md) | Reusable prompt patterns for common tasks |
| [Acceptable Use Policy](../rules/acceptable-use.md) | Full policy on what's allowed |
| [Code Standard](../../code/README.md) | The quality bar all code must meet |
| [Security Standard](../../security/README.md) | Security requirements and practices |
