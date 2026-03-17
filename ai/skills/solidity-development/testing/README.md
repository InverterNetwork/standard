# Solidity Testing Skills

This folder contains AI skills for writing Solidity unit tests following the Inverter testing methodology. Each skill covers a specific phase of the testing workflow and is designed to be used as a reference file (e.g. in Cursor) when prompting an AI assistant.

## Workflow Order

Tests are built incrementally. Each step depends on the output of the previous one, and **you should review and verify the result after each step before moving on**. AI-generated tests can be subtly wrong — catching issues early is far cheaper than debugging a broken test suite later.

### Step 1: Skeleton

**Skill:** [test-skeleton.md](./test-skeleton.md)

Create the test file structure with all sections, placeholders, and the exposed mock contract. This is the scaffold everything else gets built into.

**Output:** A compilable test file with empty sections and `/* Test: ... */` placeholders for every modifier, external function, and internal function in the contract.

> **Verify:** The skeleton compiles. Every function and modifier in the source contract has a corresponding placeholder.

---

### Step 2: Modifiers

**Skill:** [test-modifiers.md](./test-modifiers.md)

Implement modifier tests in two parts: isolation tests (testing modifier logic via exposed functions) and in-position tests (verifying modifiers are applied to each function). This step does **not** touch business logic.

**Output:** All custom modifiers have isolation tests with fuzzing. Every external function with modifiers has a `ModifierInPositionChecks` test using concrete values.

> **Verify:** Run `forge test --mc ContractName_Test`. All modifier tests pass. No functional/business logic tests were added — only modifier coverage.

---

### Step 3: Internal Functions

**Skills:** [test-internal-functions.md](./test-internal-functions.md) + [test-fuzz.md](./test-fuzz.md)

Test all internal functions via the exposed mock contract. Write Gherkin documentation first (failures before success cases), then implement. Use the fuzz testing skill for designing proper fuzz tests — bounded inputs, simulation helpers, exact assertions.

**Output:** Every internal function has failure and success tests. Fuzzed success cases with simulation helpers where appropriate.

> **Verify:** This is a critical checkpoint. Internal function tests form the foundation for everything above them. Run the tests **and review the Gherkin + implementation carefully**. Confirm:
> - Every internal function is covered
> - Failure cases test all revert paths
> - Success cases verify state changes, events, and return values
> - Fuzz bounds are realistic and constraints prevent impossible inputs
> - Simulation helpers mirror the contract logic independently (not calling the contract to verify itself)

---

### Step 4: External Functions

**Skills:** [test-external-functions.md](./test-external-functions.md) + [test-fuzz.md](./test-fuzz.md)

Test the public API. At this point modifiers and internals are already covered, so external function tests focus on integration — does the function wire things together correctly? Again, use the fuzz testing skill for proper fuzz test design.

**Output:** Every external/public function has failure and success tests. No duplication of modifier or internal function logic already tested in Steps 2–3.

> **Verify:** Another critical checkpoint. External functions are the contract's interface and where most bugs surface in integration. Run tests and review carefully:
> - No re-testing of modifier logic or internal function logic
> - Integration behaviour is correct (functions call the right internals with the right arguments)
> - State changes, events, and return values are all asserted
> - Fuzz tests use proper bounds and exact verification

---

### Step 5 (Optional): E2E Tests

**Skill:** [test-e2e.md](./test-e2e.md)

End-to-end tests verify complete workflows across multiple modules (e.g. buy → sell, role assignment → function execution). These are integration tests from a user's perspective.

> **Note:** This skill is less refined than the core unit testing skills. Expect to do more manual adjustment. Use it as a structural guide rather than a drop-in recipe.

---

### Step 6 (Optional): Invariant Tests

**Skill:** [test-invariant.md](./test-invariant.md)

Invariant tests use fuzzing to verify that critical protocol properties hold under randomized sequences of operations. They require handlers, ghost variables, and careful setup.

> **Note:** This skill is less refined than the core unit testing skills. It provides a solid starting framework, but handler design and invariant selection will likely need manual iteration.

---

## Quick Reference

| Step | Skill | Depends On | Key Concern |
|------|-------|------------|-------------|
| 1 | [test-skeleton.md](./test-skeleton.md) | Source contract | Compiles, all placeholders present |
| 2 | [test-modifiers.md](./test-modifiers.md) | Step 1 | Isolation + in-position, no business logic |
| 3 | [test-internal-functions.md](./test-internal-functions.md) + [test-fuzz.md](./test-fuzz.md) | Step 2 | Full coverage, exact fuzz assertions |
| 4 | [test-external-functions.md](./test-external-functions.md) + [test-fuzz.md](./test-fuzz.md) | Step 3 | Integration only, no duplication |
| 5 | [test-e2e.md](./test-e2e.md) | Step 4 | Multi-module workflows (less refined) |
| 6 | [test-invariant.md](./test-invariant.md) | Step 4 | Protocol-wide properties (less refined) |

## Tips

- **One step at a time.** Don't try to generate the entire test file in a single prompt. The quality degrades fast. Run one skill, verify, then move on.
- **Review after Steps 3 and 4 especially.** Internal and external function tests are where most complexity lives. Bugs in test logic here will give false confidence.
- **Fuzz testing is not optional.** The [test-fuzz.md](./test-fuzz.md) skill is used alongside Steps 3 and 4 — it's not a separate phase. Any function with arithmetic, boundaries, or variable inputs should be fuzzed.
- **Run tests after every step.** `forge test --mc ContractName_Test` should pass before you move to the next skill. Don't accumulate untested output.
- **The AI will try to skip things.** Watch for missing failure cases, lazy `assertTrue(x > 0)` instead of exact assertions, and hardcoded fuzz environments. The skills document these anti-patterns — enforce them.
