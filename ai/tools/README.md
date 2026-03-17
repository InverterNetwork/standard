# AI Tools

This section catalogues the AI tools approved for use within the Inverter Network and provides platform-specific guidance for each.

## Approved Tools

| Tool | Type | Primary Use Cases |
|------|------|-------------------|
| [Claude](./claude.md) | LLM (Anthropic) | Code generation, analysis, long-context reasoning |
| [ChatGPT](./chatgpt.md) | LLM (OpenAI) | General assistance, brainstorming, drafting |
| [Cursor](./cursor.md) | AI-powered IDE | Inline code completion, codebase-aware editing, chat-driven development |

## How to Choose

- **Writing or editing code in-editor** → Cursor
- **Deep analysis, long documents, or complex reasoning** → Claude
- **Quick questions, brainstorming, or general drafting** → ChatGPT

> Each tool guide covers setup, recommended configurations, tips, and known limitations. Refer to the individual pages for details.

## Adding a New Tool

When a new AI tool is adopted by the team:

1. Create a new markdown file in this directory (e.g. `tool-name.md`)
2. Follow the structure of existing tool guides
3. Add the tool to the table above and to the main [AI README](../README.md)
4. Get the addition reviewed and merged via PR
