# Claude

## What It Is

Large language model by Anthropic. Best-in-class for long-context reasoning, code analysis, and structured output.

## When to Use

- Deep analysis of contracts, architectures, or security considerations
- Reviewing long documents or codebases that exceed other tools' context limits
- Complex multi-step reasoning (e.g. "trace this state transition across three contracts")
- Drafting structured artifacts (specs, documentation, proposals)

When to reach for something else:
- Writing or editing code in-editor → [Cursor](./cursor.md)
- Quick one-off questions or brainstorming → [ChatGPT](./chatgpt.md)

## Setup

Use the latest available model (currently Claude Sonnet 4 / Opus 4) for best results. When using the API, prefer the highest-capability model your plan allows.

**Projects** — Use Claude's Projects feature to pin frequently referenced files (code standard, security standard, interface definitions) as persistent context.

## Tips

- Claude handles large context well. When analysing a contract, include the full source + interface + relevant dependencies rather than excerpts.
- For code review, paste the full diff and ask for specific concerns (security, naming, structure) rather than a generic "review this".
- Claude respects structured instructions. If you want output in a specific format, describe the format explicitly.
- When asking about Inverter conventions, include the relevant standard section as context — Claude won't know Inverter-specific rules without it.

## Limitations

- Does not have access to your codebase or git history unless you paste it in.
- Can hallucinate function signatures, error names, and import paths. Always verify generated code compiles.
- Context window is large but not infinite. For very large codebases, be selective about what you include.
- Check your plan's data retention policy. API usage with data retention disabled is preferred for anything sensitive.

## Links

- [claude.ai](https://claude.ai)
- [API Documentation](https://docs.anthropic.com)
- [Usage Policy](https://www.anthropic.com/policies)
