# Cursor

## What It Is

AI-powered code editor built on VS Code. Provides codebase-aware completions, inline editing, and a chat/agent interface that can read and modify files directly.

## When to Use

- Writing, editing, or refactoring code in the editor
- Tasks that require codebase context (the AI can read your project files)
- Multi-file changes where the AI needs to understand project structure
- Running skills from the Inverter AI standard as part of development

When to reach for something else:
- Deep analysis of long documents or specs → [Claude](./claude.md)
- Quick questions unrelated to the current codebase → [ChatGPT](./chatgpt.md)

## Setup

1. Download from [cursor.com](https://cursor.com) and install
2. Open your Inverter project repository
3. Cursor indexes your codebase automatically for context-aware responses

**Model Selection** — Use the most capable model available for complex tasks (agent mode, multi-file changes). Faster models are fine for tab completions and simple questions.

**Rules** — Cursor supports project-level rules (`.cursor/rules/`) that persist AI guidance across sessions. Use these for Inverter-specific conventions (naming, structure, testing patterns) so the AI follows them without being told each time.

**Skills** — Reference skill files from the `ai/skills/` folder as context when prompting. For example, attach the relevant testing skill when generating test code.

## Tips

- **Agent mode** (`Cmd/Ctrl + I`) is best for multi-step tasks — it can create files, run commands, and iterate. Use it for scaffolding, test generation, and refactoring.
- **Chat mode** (`Cmd/Ctrl + L`) is better for questions and analysis where you don't want the AI to edit files.
- **Reference files with @** — Use `@filename` to pull specific files into context. This is more reliable than hoping the AI finds the right file on its own.
- **Attach the standard** — When doing work that should follow Inverter conventions (scaffolding a contract, writing tests, generating NatSpec), attach the relevant section of the code standard or the appropriate skill as context.
- **Review every change** — Cursor makes it easy to accept changes quickly. Resist the urge. Read diffs carefully, especially for Solidity where subtle bugs have real consequences.

## Limitations

- Context window is shared between your prompt, referenced files, and codebase indexing. For very large files or many references, the AI may miss details at the edges.
- Tab completions are fast but shallow — they don't understand full project architecture. Use agent/chat for anything non-trivial.
- Generated code follows general Solidity patterns but not Inverter-specific conventions unless you provide rules or skill files as context.
- Cursor runs locally but sends prompts to cloud-hosted models. The [Acceptable Use Policy](../rules/acceptable-use.md) applies — never include secrets in prompts or referenced files.

## Links

- [cursor.com](https://cursor.com)
- [Documentation](https://docs.cursor.com)
- [Privacy Policy](https://www.cursor.com/privacy)
