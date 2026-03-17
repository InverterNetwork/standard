# Commit Messages

## Overview

Generate well-formatted commit messages for Inverter Network repositories. The AI should inspect the actual staged changes via git rather than relying on a pasted summary.

## Format

```
<Keyword>: <Short Description>
```

- **Title**: 72 characters max
- **Description** (optional): Additional context below a blank line if needed

## Keywords

| Keyword | When to use |
|---------|-------------|
| `Feat` | New feature added or extended |
| `Fix` | Bug fix or issue resolution |
| `Docs` | Documentation changes |
| `CI` | CI/CD pipeline changes |
| `Refactor` | Code refactoring (no behaviour change) |
| `Test` | Test changes |
| `Script` | Script changes |
| `Revert` | Reverting an existing commit |

## Workflow

### Step 1: Inspect the Changes Directly

Do not ask the user to describe their changes. Look at them.

```bash
# Staged changes (what will be committed)
git diff --staged

# If nothing is staged, check unstaged changes
git diff

# For broader context: what files changed
git status

# Recent commits for style reference
git log --oneline -10
```

Read the diff carefully. Understand **what** changed and **why** — not just which lines moved.

### Step 2: Classify the Change

Determine the keyword by asking:

- Did new functionality get added? → `Feat`
- Was a bug or incorrect behaviour fixed? → `Fix`
- Did only docs/comments/READMEs change? → `Docs`
- Is it a structural change with no behaviour difference? → `Refactor`
- Did only tests change? → `Test`
- Did only deployment/utility scripts change? → `Script`
- Did only CI/CD config change? → `CI`

If a commit spans multiple categories (e.g. a feature + its tests), use the primary category — typically `Feat` or `Fix`.

### Step 3: Write the Message

Focus on **why**, not **what**. The diff already shows what changed.

**Good:**
```
Feat: Add loan consolidation for multi-position borrowers
```

**Bad:**
```
Feat: Add consolidateLoans function to CreditFacility_v1 and update tests
```

The first one tells you the purpose. The second one just narrates the diff.

### Step 4: Check the Rules

- [ ] Title is 72 characters or less
- [ ] Starts with an approved keyword followed by `: `
- [ ] No issue numbers, PR numbers, or branch names in the title
- [ ] Description (if present) is separated by a blank line
- [ ] Message is in English, professional tone

## Examples

**Single feature:**
```
Feat: Add role-based access control to payment processing
```

**Bug fix:**
```
Fix: Prevent double-spending when loans are transferred mid-repayment
```

**Refactor:**
```
Refactor: Extract fee calculation into shared internal function
```

**Docs:**
```
Docs: Add NatSpec to all public functions in BountyManager_v1
```

**With description:**
```
Feat: Add configurable loop cap for leveraged positions

Introduces a maxLoops parameter that limits recursive borrowing depth.
Defaults to 5 and is adjustable via permissioned setter.
```

## Tips

- **One concern per commit.** If the diff touches unrelated things, suggest splitting.
- **Read the recent log first.** Match the tense and style the team already uses.
- **Don't over-explain.** If the title is clear, a description is optional.
- **Watch for mixed categories.** A `Feat` that also fixes a bug is still `Feat` unless the fix is the primary intent.
