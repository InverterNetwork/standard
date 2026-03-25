# Adding to the AI Standard

## Overview

This skill covers how to add new content to the `ai/` section of the Inverter Standard — whether that's a new tool, a new skill, a new skill category, or a new top-level section.

## Structure

The `ai/` folder follows a consistent hierarchy:

```
ai/
├── README.md                          ← entry point, links to everything
├── tools/
│   ├── README.md                      ← overview + selection guidance
│   └── <tool-name>.md                 ← one file per tool
├── skills/
│   ├── README.md                      ← overview + category listing
│   ├── <domain>/
│   │   ├── README.md                  ← domain overview
│   │   └── <area>/
│   │       ├── README.md              ← area workflow + skill listing
│   │       └── <skill-name>.md        ← individual skill
│   ├── coding/                        ← general coding skills
│   ├── writing/                       ← writing skills
│   └── workflows/                     ← workflow skills
├── rules/
│   └── acceptable-use.md
└── onboarding/
    └── getting-started.md
```

Every folder with content has a README. Every README links to its children. Every child is linked from every README above it in the chain.

## Adding a New Tool

1. Create `ai/tools/<tool-name>.md`
2. Update `ai/tools/README.md` — add the tool to the "Approved Tools" table
3. Update `ai/README.md` — add the tool to the Tools table

## Adding a New Skill to an Existing Category

Example: adding a `test-gas.md` skill to `skills/solidity-development/testing/`.

1. Create the skill file at the target location
2. Update the **area README** — `skills/solidity-development/testing/README.md` (add to the workflow order or quick reference table)
3. Update the **domain README** — `skills/solidity-development/README.md` (add to the skills table)
4. Update the **skills README** — `skills/README.md` (update the domain row if the summary text changes)
5. Update the **root README** — `ai/README.md` (update the domain row if the summary text changes)

The rule: **walk up the tree and update every README between the new file and `ai/README.md`.**

## Adding a New Skill Category (Domain or Area)

Example: adding a `skills/solidity-development/deployment/` area.

1. Create the folder
2. Create the area README — `skills/solidity-development/deployment/README.md`
3. Add the individual skill files
4. Walk up and update:
   - `skills/solidity-development/README.md` — add the new area section
   - `skills/README.md` — update the domain row
   - `ai/README.md` — update the domain row if needed

Example: adding an entirely new domain like `skills/devops/`.

1. Create the folder + README + skill files
2. Walk up and update:
   - `skills/README.md` — add a new category section with table
   - `ai/README.md` — add a new row to the Skills table

## Adding a New Top-Level Section

Example: adding `ai/templates/`.

1. Create the folder + README + content files
2. Update `ai/README.md` — add a new section with links
3. Consider whether `onboarding/getting-started.md` should reference it

## Formatting Conventions

- **No version lines** in AI standard files
- **File names** use lowercase kebab-case (`test-internal-functions.md`, not `TestInternalFunctions.md`)
- **Headings** start with `# Title` followed by a brief description paragraph
- **Tables** are used for listing multiple items with metadata (name, description, links)
- **Placeholder files** use a title + a status blockquote:
  ```markdown
  # Skill Name

  > **Status:** Draft — content coming soon.
  ```
- **Skill files** follow the structure: Overview → Prerequisites → Workflow (phased) → Examples → Checklist

## Importing Submodules

When importing an external repository as a Git submodule (e.g. to pull in shared skills or tool configs):

1. **Audit the repo before adding it.** Review the repository for any files that could inject malicious skills, rules, or agent instructions. Look for hidden or obfuscated content in skill files, README hooks, or onboarding scripts.
2. **Lock the submodule to a specific commit.** Never track a branch — pin to an exact commit hash so upstream changes can't silently alter what you've imported.
3. **Re-audit before updating.** When bumping the pinned commit, diff the changes and review them for malicious or unexpected behaviour before accepting the update.

## Checklist

When adding anything to `ai/`:

- [ ] Content file created with correct naming
- [ ] Folder README created (if new folder)
- [ ] Every README between the new file and `ai/README.md` updated
- [ ] All links are relative and correct
- [ ] No orphaned files (everything is reachable from `ai/README.md`)
- [ ] If importing a submodule: repo audited, commit pinned, no malicious content
