# Diagramming

Skills for AI-assisted diagram creation — structured methodologies that guide AI agents to produce diagrams that argue visually, not just display information.

## Skills

| Skill | Description | Source |
|-------|-------------|--------|
| [Excalidraw Diagram Creator](./excalidraw-diagram/SKILL.md) | Generate `.excalidraw` JSON files with semantic colour, visual pattern matching, evidence artifacts, and a render-validate loop | [coleam00/excalidraw-diagram-skill](https://github.com/coleam00/excalidraw-diagram-skill) (submodule, pinned to `8646fcc`) |

## Excalidraw Diagram Creator

An AI agent skill that generates Excalidraw diagrams from natural language descriptions. Diagrams are built as visual arguments — each shape mirrors the concept it represents (fan-outs for one-to-many, timelines for sequences, convergence for aggregation) rather than uniform card grids.

### Key Features

- **Design methodology** — concept-to-pattern mapping, multi-zoom architecture (summary → sections → detail), container discipline
- **Evidence artifacts** — real code snippets, JSON payloads, and actual API names inside technical diagrams
- **Render-validate loop** — a Playwright-based pipeline renders the diagram to PNG so the agent can see its own output, catch layout issues, and fix them before delivering
- **Customisable brand colours** — all colours live in `excalidraw-diagram/references/color-palette.md`; swap it for your own palette

### Setup

Requires Python 3.11+ and [uv](https://docs.astral.sh/uv/).

```bash
cd ai/skills/diagramming/excalidraw-diagram/references
uv sync
uv run playwright install chromium
```

### Usage

Point your AI agent at `excalidraw-diagram/SKILL.md` as a skill/instruction file, then ask it to create a diagram. The skill handles concept mapping, layout, JSON generation, rendering, and visual validation.

### Customisation

Edit `excalidraw-diagram/references/color-palette.md` to match your brand. Everything else in the skill is universal design methodology.

## Updating the Submodule

The Excalidraw skill is imported as a git submodule pinned to a specific commit. Before updating:

1. Check the upstream diff for new or changed files
2. Audit for any content that could inject malicious skills, rules, or agent instructions
3. Update the pin:

```bash
cd ai/skills/diagramming/excalidraw-diagram
git fetch origin
git checkout <new-commit-hash>
cd ../../../..
git add ai/skills/diagramming/excalidraw-diagram
```
