---
description: Comprehensive guide for writing in-depth fuzz tests
globs:
  - "test/**/*.t.sol"
---

# Fuzz Testing Workflow

## Overview

Fuzz tests verify that functions behave correctly across a wide range of inputs. Unlike concrete tests with fixed values, fuzz tests explore the input space to find edge cases and bugs that manual testing misses.

**Core Philosophy: NO SHORTCUTS**

Fuzz testing requires the same rigor as any other testing—more so, because the inputs vary. Two principles govern everything:

1. **Environment**: If state can be fuzzed, fuzz it. If bounds can be derived, derive them.
2. **Verification**: If the math is deterministic, the assertion MUST be exact.

**Testing Strategy:**

1. **Identify** - Which functions benefit from fuzzing
2. **Design** - Define the input space with proper bounds
3. **Build** - Create fuzzed environments with helpers (NO SHORTCUTS)
4. **Write** - Implement tests with simulation helpers
5. **Verify** - Assert exact expected values (NO SHORTCUTS)
6. **Debug** - Run, interpret failures, fix

## Prerequisites

- Test file exists with proper structure
- Understanding of the function's domain constraints
- Exposed mock contracts for internal function testing
- Knowledge of realistic value ranges for the domain

## High-Level Workflow

### Phase 1: Identify Fuzz Candidates

Not all functions need fuzzing. Identify candidates based on:

#### Good Candidates for Fuzzing

- **Calculations**: Any function with arithmetic (especially division, multiplication)
- **State transitions**: Functions that change state based on input amounts
- **Boundary conditions**: Functions with min/max limits, thresholds
- **Multi-input functions**: Where inputs interact (e.g., loops × collateral)
- **Array/loop logic**: Functions iterating over variable-length data

#### Less Suitable for Fuzzing

- Simple getters returning storage values
- Boolean toggles with no arithmetic
- Functions with only 1-2 valid input states

#### Step 1: Analyze the Function

For each fuzz candidate, identify:

- **Input parameters**: What are the types and realistic ranges?
- **Constraints**: What must be true for valid inputs?
- **Interdependencies**: Do inputs constrain each other?
- **Expected outputs**: What should the function return/emit/change?
- **Edge cases**: Zero values, max values, boundaries?

### Phase 2: Design Input Space

Proper input bounding is critical. Poor bounds waste fuzzer cycles on invalid inputs or miss important ranges.

#### Basic Bounding with `bound()`

Use `bound()` to constrain fuzz inputs to valid ranges:

```solidity
function testFuzz_Calculate(uint amount_, uint price_) public {
    // Bound to realistic DeFi ranges
    amount_ = bound(amount_, 1e18, 1e27);      // 1 to 1 billion tokens
    price_ = bound(price_, 0.01e18, 10000e18); // $0.01 to $10,000

    // ... test logic
}
```

#### Using `vm.assume()` for Constraints

Use `vm.assume()` for constraints that can't be expressed with `bound()`:

```solidity
function testFuzz_Divide(uint numerator_, uint denominator_) public {
    numerator_ = bound(numerator_, 1, type(uint128).max);
    denominator_ = bound(denominator_, 1, type(uint128).max);

    // Constraint: result must be non-zero
    vm.assume(numerator_ >= denominator_);

    uint result = _contract.divide(numerator_, denominator_);
    assertGe(result, 1);
}
```

#### Overflow Prevention

Prevent arithmetic overflow in fuzzed calculations:

```solidity
function testFuzz_Multiply(uint a_, uint b_) public {
    // Bound to prevent overflow: a * b <= type(uint256).max
    a_ = bound(a_, 1, type(uint128).max);
    b_ = bound(b_, 1, type(uint128).max);

    // Additional overflow check if needed
    vm.assume(a_ <= type(uint256).max / b_);

    uint result = _contract.multiply(a_, b_);
    assertEq(result, a_ * b_);
}
```

#### Address Handling

Filter addresses properly:

```solidity
function testFuzz_Transfer(address to_, uint amount_) public {
    // Filter invalid addresses
    vm.assume(to_ != address(0));
    vm.assume(to_.code.length == 0);  // EOA only, no contracts
    vm.assume(to_ != address(this));   // Not self
    vm.assume(uint160(to_) > 10);      // Skip precompiles (1-9)

    amount_ = bound(amount_, 1, type(uint128).max);

    // ... test logic
}
```

#### DeFi-Specific Ranges

Common realistic ranges for DeFi testing:

```solidity
// Token amounts (18 decimals)
uint constant MIN_AMOUNT = 1e18;           // 1 token
uint constant MAX_AMOUNT = 1e27;           // 1 billion tokens

// Prices (18 decimals)
uint constant MIN_PRICE = 0.001e18;        // $0.001
uint constant MAX_PRICE = 100_000e18;      // $100,000

// Percentages (basis points)
uint constant MIN_BPS = 1;                 // 0.01%
uint constant MAX_BPS = 10_000;            // 100%

// Loops
uint constant MIN_LOOPS = 1;
uint constant MAX_LOOPS = 10;

// Time
uint constant MIN_DURATION = 1 hours;
uint constant MAX_DURATION = 365 days;
```

#### Interdependent Bounds

When one input constrains another, derive bounds from fuzzed state:

```solidity
function testFuzz_BorrowWithCollateral(
    uint collateral_,
    uint borrowRatio_
) public {
    collateral_ = bound(collateral_, 1e18, 1e27);
    borrowRatio_ = bound(borrowRatio_, 1, 80);  // 1% to 80% LTV

    // Derive borrow amount FROM collateral (interdependent)
    uint maxBorrow = (collateral_ * 80) / 100;  // 80% max LTV
    uint borrowAmount = (collateral_ * borrowRatio_) / 100;

    vm.assume(borrowAmount > 0);
    vm.assume(borrowAmount <= maxBorrow);

    // ... test logic
}
```

### Phase 3: Build Fuzzed Environments (NO SHORTCUTS)

Environment creation must be as rigorous as verification. No hardcoded shortcuts.

#### BAD: Environment Shortcuts (FORBIDDEN)

```solidity
// FORBIDDEN: Hardcoded segments instead of fuzzed
PackedSegment[] memory segments = new PackedSegment[](2);
segments[0] = helper_createSegment(1e18, 0, 100e18, 1);  // Always same
segments[1] = helper_createSegment(2e18, 0.1e18, 50e18, 10);  // Always same

// FORBIDDEN: Magic numbers not derived from state
uint buyAmount = 1000e18;  // Where does this come from?

// FORBIDDEN: Skipping complex setup
// "Just use a simple case, it's good enough" - NO!
```

#### GOOD: Proper Fuzzed Environment

```solidity
function testFuzz_RaiseFloor(
    uint floorPrice_,
    uint floorSupply_,
    uint premiumCountSeed_,
    uint seed_,
    uint buyFuzz_
) public {
    // Bound base parameters
    floorPrice_ = bound(floorPrice_, 0.01e18, 1000e18);
    floorSupply_ = bound(floorSupply_, 1e18, 1e27);
    uint premiumCount = bound(premiumCountSeed_ % 10, 1, 5);

    // Create fuzzed environment using helper
    PackedSegment[] memory segments = helper_createFuzzedFloorSegments(
        floorPrice_, floorSupply_, premiumCount, seed_
    );
    floorContract.reconfigureSegments(segments, 0, true);

    // Derive constraints FROM the fuzzed state
    (uint p1,, uint s1, uint n1) = segments[1]._unpack();
    vm.assume(n1 >= 1);
    uint costStep0 = helper_calculateStepCost(p1, s1);

    uint maxCollateral = helper_getMaxCollateralForIncrease(segments);
    vm.assume(maxCollateral >= costStep0);

    // Bound dependent values using derived constraints
    uint increaseCollateral = costStep0;
    uint maxBuyCap = maxCollateral - increaseCollateral;
    uint buyAmount = bound(buyFuzz_, 1, maxBuyCap);

    // Prime the environment
    helper_primeWithBuy(buyAmount, increaseCollateral);

    // ... test execution and verification
}
```

#### Fuzzed Environment Builder Pattern

Create `helper_createFuzzed*()` functions for complex state:

```solidity
/// @notice Create fuzzed floor segments with seed-based variety
/// @param floorPrice_ Base floor price (fuzzed)
/// @param floorSupply_ Base floor supply (fuzzed)
/// @param premiumCount_ Number of premium segments
/// @param seed_ Seed for deterministic variety
function helper_createFuzzedFloorSegments(
    uint floorPrice_,
    uint floorSupply_,
    uint premiumCount_,
    uint seed_
) internal pure returns (PackedSegment[] memory) {
    PackedSegment[] memory segments = new PackedSegment[](1 + premiumCount_);

    // Floor segment (always valid structure)
    segments[0] = helper_createSegment(floorPrice_, 0, floorSupply_, 1);

    // Premium segments (fuzzed using seed)
    uint prevFinalPrice = floorPrice_;
    for (uint i = 0; i < premiumCount_; i++) {
        // Derive unique seed for each segment (deterministic)
        uint segSeed = uint(keccak256(abi.encode(seed_, i)));

        // Calculate segment parameters from seed
        uint initialPrice = prevFinalPrice + ((segSeed % 1e18) + 0.01e18);
        uint supplyPerStep = bound((segSeed >> 16), 1e18, 1e27);
        uint steps = bound((segSeed >> 24), 1, 100);

        // Price increase based on step count (domain constraint)
        uint priceIncrease;
        if (steps == 1) {
            priceIncrease = 0;  // Single step must be flat
        } else {
            priceIncrease = bound((segSeed >> 8), 0.001e18, 2e18);
        }

        segments[i + 1] = helper_createSegment(
            initialPrice, priceIncrease, supplyPerStep, steps
        );
        prevFinalPrice = initialPrice + (priceIncrease * (steps - 1));
    }

    return segments;
}
```

#### Seed-Based Determinism

Use seeds to create reproducible variety across array elements:

```solidity
// Generate unique but deterministic values for each element
for (uint i = 0; i < count; i++) {
    uint elementSeed = uint(keccak256(abi.encode(baseSeed_, i)));

    // Extract different "random" values from the seed
    uint value1 = bound(elementSeed, MIN, MAX);
    uint value2 = bound(elementSeed >> 8, MIN2, MAX2);
    uint value3 = bound(elementSeed >> 16, MIN3, MAX3);

    // Use values...
}
```

#### Derived Constraint Helpers

Create helpers that calculate bounds from fuzzed state:

```solidity
/// @notice Calculate maximum safe collateral for floor increase
/// @param segments_ Current segment configuration (fuzzed)
/// @return maxCollateral_ Maximum collateral that maintains valid state
function helper_getMaxCollateralForIncrease(
    PackedSegment[] memory segments_
) internal pure returns (uint maxCollateral_) {
    require(segments_.length >= 2, "Need at least 2 segments");

    // Calculate based on what can be absorbed while maintaining invariants
    uint segmentsToAbsorb = segments_.length - 2;

    if (segmentsToAbsorb == 0) {
        // Only floor + 1 premium: can only absorb partial steps
        (uint price, uint priceInc, uint supply, uint steps) =
            segments_[1]._unpack();
        if (steps <= 1) return 0;

        uint stepsToAbsorb = steps - 1;
        for (uint i = 0; i < stepsToAbsorb; i++) {
            maxCollateral_ += ((price + (i * priceInc)) * supply) / 1e18;
        }
    } else {
        // Can absorb full segments plus partial
        for (uint i = 1; i <= segmentsToAbsorb; i++) {
            (uint price, uint priceInc, uint supply, uint steps) =
                segments_[i]._unpack();
            for (uint j = 0; j < steps; j++) {
                maxCollateral_ += ((price + (j * priceInc)) * supply) / 1e18;
            }
        }
        // ... handle partial steps from next segment
    }
}
```

#### Mocking Internal Dependencies for Isolation

When a function under test calls complex internal functions, use access-mocks to isolate the function's logic. This allows testing the orchestration/control flow separately from the internal implementation.

**Why isolate with mocks:**

- Test the function's logic without being affected by internal function complexity
- Verify the function calls internals with correct parameters
- Control internal return values to test different scenarios
- Faster test execution (skip expensive internal computations)

**Pattern: Mock switches with controlled return values**

In your exposed mock contract, add switches to enable mocking:

```solidity
// In the exposed mock contract
contract Floor_v1_Exposed is Floor_v1 {
    // Mock control flags
    bool public mockSegmentAbsorption;
    bool public mockReconfigureSegments;

    // Mock return values (can be set by test)
    AbsorptionResult public mockAbsorptionResult;

    // Call tracking
    uint public segmentAbsorptionCallCount;
    uint public reconfigureSegmentsCallCount;
    uint public lastReconfigureCollateral;

    // Setters for mock control
    function setMockSegmentAbsorption(bool enabled_) external {
        mockSegmentAbsorption = enabled_;
    }

    function setMockReconfigureSegments(bool enabled_) external {
        mockReconfigureSegments = enabled_;
    }

    function setMockAbsorptionResult(AbsorptionResult memory result_) external {
        mockAbsorptionResult = result_;
    }

    function resetCallCounts() external {
        segmentAbsorptionCallCount = 0;
        reconfigureSegmentsCallCount = 0;
        lastReconfigureCollateral = 0;
    }

    // Override internal function to use mock when enabled
    function _absorbPremiumSteps(uint collateral_)
        internal
        override
        returns (AbsorptionResult memory)
    {
        segmentAbsorptionCallCount++;

        if (mockSegmentAbsorption) {
            return mockAbsorptionResult;  // Return controlled value
        }
        return super._absorbPremiumSteps(collateral_);  // Real implementation
    }
}
```

**Test helper to setup mocked environment:**

```solidity
/// @notice Helper to setup mocked raiseFloor tests
/// @dev Enables mocks and resets call counts for isolated testing
function helper_setupMockedRaiseFloor() internal {
    floorContract.setMockSegmentAbsorption(true);
    floorContract.setMockReconfigureSegments(true);
    floorContract.resetCallCounts();
}
```

**Using mocks in fuzz tests:**

```solidity
/* Test: raiseFloor orchestration (isolated)
    Property: raiseFloor correctly orchestrates absorption and reconfiguration
              regardless of the specific absorption result.

    Given: Mocked internal functions with fuzzed return values
    When: raiseFloor is called
    Then: It calls absorption exactly once
    And: It calls reconfigure with the absorption result
    And: State is updated based on mock return values
*/
function testFuzz_RaiseFloor_Orchestration(
    uint collateral_,
    uint mockNewFloorPrice_,
    uint mockNewFloorSupply_,
    uint mockSegmentIndex_
) public {
    // Bound fuzz inputs
    collateral_ = bound(collateral_, 1e18, 1e24);
    mockNewFloorPrice_ = bound(mockNewFloorPrice_, 0.01e18, 1000e18);
    mockNewFloorSupply_ = bound(mockNewFloorSupply_, 1e18, 1e27);
    mockSegmentIndex_ = bound(mockSegmentIndex_, 1, 5);

    // Setup: configure mock to return fuzzed values
    AbsorptionResult memory mockResult = AbsorptionResult({
        newFloorPrice: mockNewFloorPrice_,
        newFloorSupply: mockNewFloorSupply_,
        lastConsumedSegmentIndex: mockSegmentIndex_,
        stepsConsumedFromLastSegment: 1,
        collateralUsed: collateral_
    });
    floorContract.setMockAbsorptionResult(mockResult);

    // Enable mocks
    helper_setupMockedRaiseFloor();

    // Setup minimal valid state for the function to execute
    helper_setupMinimalFloorState();

    // Execute
    floorContract.raiseFloor(collateral_);

    // Verify orchestration (exact assertions)
    assertEq(floorContract.segmentAbsorptionCallCount(), 1,
        "Absorption called exactly once");
    assertEq(floorContract.reconfigureSegmentsCallCount(), 1,
        "Reconfigure called exactly once");

    // Verify state reflects mock return values (exact assertions)
    PackedSegment floorSeg = floorContract.getFloorSection();
    (uint newPrice,, uint newSupply,) = floorSeg._unpack();
    assertEq(newPrice, mockNewFloorPrice_, "Floor price matches mock result");
    assertEq(newSupply, mockNewFloorSupply_, "Floor supply matches mock result");
}
```

**When to use mocked isolation:**

- Function orchestrates multiple complex internal functions
- Internal functions have their own dedicated tests
- You want to test control flow and parameter passing
- Full integration test would be too slow or complex to setup

**When NOT to use mocked isolation:**

- Simple functions with no internal dependencies
- You need to verify end-to-end behavior (use integration tests)
- The internal function behavior IS the property being tested

**Combining isolated and integration tests:**

```solidity
// Isolated test: verify orchestration with mocks
function testFuzz_RaiseFloor_Orchestration(...) public { }

// Integration test: verify full behavior without mocks
function testFuzz_RaiseFloor_SingleStep_Integration(...) public { }
function testFuzz_RaiseFloor_MultipleSteps_Integration(...) public { }
```

### Phase 4: Write Fuzz Tests

#### Place Tests in the Correct Section

Fuzz tests must be placed in the correct section of the test file based on what function they're testing. The test file structure mirrors the contract structure:

```solidity
///////////////////////////////////////////////////////////////////////////
// Test Modifiers
// ...

///////////////////////////////////////////////////////////////////////////
// Test External Functions
//
// Public Functions - Getter Functions
// Public Functions - Mutating Functions
// ...

///////////////////////////////////////////////////////////////////////////
// Test Internal Functions
//
// Internal - Calculations
// Internal - State Management
// Internal - Token Operations
// ...

///////////////////////////////////////////////////////////////////////////
// Helper Functions
// ...
```

**Placement rules:**

- **Testing an internal function** (via `exposed_*`) → Place in "Test Internal Functions" section, in the appropriate subsection
- **Testing an external/public function** → Place in "Test External Functions" section
- **Testing a modifier** → Place in "Test Modifiers" section

**Example:** If you're fuzz testing `_absorbPremiumSteps` (internal), place it in:

```solidity
///////////////////////////////////////////////////////////////////////////
// Test Internal Functions

//--------------------------------------------------------------------------
// Internal - Absorption

/* Test: _absorbPremiumSteps (single step)
    Property: ...
*/
function testFuzz_AbsorbPremiumSteps_SingleStep(...) public {
    // Uses exposed_absorbPremiumSteps
}
```

**NOT** in the external functions section, even if `raiseFloor` (external) calls it.

**Why this matters:**

- Maintains consistent test file organization
- Makes it easy to find tests for specific functions
- Matches the skeleton structure from test-skeleton.mdc
- Keeps internal function tests separate from integration tests

#### Document What You're Testing

Before writing any fuzz test, clearly articulate the property or behavior being tested. Use a structured comment block above the test function.

**Required documentation:**

```solidity
/* Test: raiseFloor (single step absorption)
    Property: When collateral equals exactly one premium step cost,
              exactly one step is absorbed into the floor.

    Given: A fuzzed curve with floor + premium segments
    And: Collateral equals the exact cost of the first premium step
    And: Buy is primed to enable floor raising
    When: raiseFloor is called with that collateral amount
    Then: Exactly one step is absorbed from the first premium segment
    And: Floor price equals that step's price
    And: Floor supply increases by that step's supply
    And: Reconfigure is called exactly once
*/
function testFuzz_RaiseFloor_SingleStep(
    uint floorPrice_,
    uint floorSupply_,
    uint premiumCountSeed_,
    uint seed_,
    uint buyFuzz_
) public {
    // Implementation...
}
```

**What to include:**

1. **Property statement**: The core invariant or behavior being verified
2. **Given/When/Then**: Structured description of setup, action, and expected outcome
3. **Exact expectations**: What specific values/states should result

**Why this matters:**

- Forces you to think about WHAT you're testing before HOW
- Makes test intent clear to reviewers
- Helps identify if bounds and constraints are sufficient
- Documents the expected behavior for future maintenance

**BAD: Missing or vague documentation**

```solidity
// FORBIDDEN: No documentation
function testFuzz_RaiseFloor(uint amount_) public { }

// FORBIDDEN: Vague documentation
/* Test raiseFloor works */
function testFuzz_RaiseFloor(uint amount_) public { }

// FORBIDDEN: Documentation doesn't match test
/* Tests single step absorption */
function testFuzz_RaiseFloor_MultipleSteps(...) public { }  // Name says multiple!
```

#### Function Naming Convention

```solidity
// Pattern: testFuzz_FunctionName_PropertyOrScenario
function testFuzz_CalculatePurchaseReturn_ValidAmounts() public { }
function testFuzz_RaiseFloor_SingleStep() public { }
function testFuzz_CreateLoan_VariousLoops() public { }
```

#### Parameter Conventions

- All fuzz parameters end with `_` suffix
- Use descriptive names indicating the domain

```solidity
function testFuzz_Buy(
    uint depositAmount_,
    uint minAmountOut_,
    address buyer_
) public {
    // ...
}
```

#### Prevent Impossible Combinations (Don't Early Return)

Early returns are a symptom of lazy input design. If you find yourself writing `if (condition) return;`, ask: **why didn't I prevent this combination in the first place?**

**BAD: Early returns for impossible combinations (LAZY)**

```solidity
function testFuzz_ComplexOperation(uint a_, uint b_, uint c_) public {
    a_ = bound(a_, 1, 1000);
    b_ = bound(b_, 1, 1000);
    c_ = bound(c_, 1, 1000);

    // LAZY: We didn't think about constraints upfront
    if (a_ + b_ < c_) return;  // Wasted fuzz cycle!

    // LAZY: Helper returns false for invalid state
    if (!helper_buildValidState(a_, b_, c_)) {
        return;  // Another wasted fuzz cycle!
    }
}
```

**GOOD: Design constraints so impossible combinations never arise**

```solidity
function testFuzz_ComplexOperation(uint a_, uint b_, uint cRatio_) public {
    a_ = bound(a_, 1, 1000);
    b_ = bound(b_, 1, 1000);

    // PROPER: Derive c from a and b so triangle inequality always holds
    // c must be < a + b, so bound cRatio to [1, 99] percent of (a + b)
    cRatio_ = bound(cRatio_, 1, 99);
    uint c = ((a_ + b_) * cRatio_) / 100;

    // Now c < a + b is GUARANTEED - no early return needed

    // PROPER: Helper always succeeds because inputs are valid by construction
    ComplexState memory state = helper_buildValidState(a_, b_, c);

    // Continue with test - 100% of fuzz cycles are useful
}
```

**The principle:** If your bounds and helpers are designed correctly, every fuzz input combination should be valid. Early returns indicate incomplete thinking about:

1. **Input relationships** - Should one input be derived from another?
2. **Bound ranges** - Are bounds too wide? Should they be tighter?
3. **Helper design** - Does the helper handle all valid input ranges?

**When early returns are acceptable (rare):**

- External dependencies with truly unpredictable behavior
- Extremely complex constraint systems where perfect bounds are impractical
- Must document WHY the early return exists and create a TODO to improve

```solidity
// ACCEPTABLE (with documentation):
// Early return due to external oracle returning stale price
// TODO(#123): Mock oracle to eliminate this early return
if (oraclePrice == 0) return;
```

### Phase 5: Exact Verification (NO SHORTCUTS)

This is the most critical phase. Fuzz tests MUST verify exact expected values.

#### BAD: Verification Shortcuts (FORBIDDEN)

```solidity
// FORBIDDEN: Lazy relative checks
assertTrue(newPrice >= p1, "Floor price >= step price");
// ^ Could be any value >= p1, doesn't verify correctness

// FORBIDDEN: Vague "something changed" assertions
assertTrue(newSupply >= oldFloorSupply, "Floor supply increased");
// ^ Increased by how much? By the right amount?

// FORBIDDEN: Non-zero checks when exact value is known
assertGt(result, 0, "Result is positive");
// ^ What's the actual expected value?

// FORBIDDEN: Range checks when exact value is calculable
assertGe(fee, minFee);
assertLe(fee, maxFee);
// ^ What's the exact fee? Calculate it!
```

#### GOOD: Exact Verification with Simulation Helpers

```solidity
function testFuzz_RaiseFloor_SingleStep(
    uint floorPrice_,
    uint floorSupply_,
    uint premiumCountSeed_,
    uint seed_,
    uint buyFuzz_
) public {
    // ... environment setup (see Phase 3) ...

    // BEFORE calling function: pre-calculate exact expected values
    (uint oldFloorPrice,, uint oldFloorSupply,) = segments[0]._unpack();
    AbsorptionResult memory exp = helper_simulateAbsorption(
        segments, oldFloorPrice, oldFloorSupply, increaseCollateral
    );

    // Verify simulation gives expected single-step result
    assertEq(exp.lastConsumedSegmentIndex, 1, "Should consume from segment 1");
    assertEq(exp.stepsConsumedFromLastSegment, 1, "Should consume exactly 1 step");
    assertEq(exp.newFloorPrice, p1, "New floor price should equal step price");
    assertEq(exp.newFloorSupply, oldFloorSupply + s1, "Supply should increase by step supply");

    // Execute the function under test
    floorContract.raiseFloor(increaseCollateral);

    // AFTER: Assert exact equality with pre-calculated expected values
    PackedSegment floorSeg = floorContract.getFloorSection();
    (uint newPrice,, uint newSupply, uint steps) = floorSeg._unpack();

    assertEq(newPrice, exp.newFloorPrice, "Floor price matches expected");
    assertEq(newSupply, exp.newFloorSupply, "Floor supply matches expected");
    assertEq(steps, 1, "Floor has exactly 1 step");

    // Verify call counts if using mocks
    assertEq(floorContract.reconfigureSegmentsCallCount(), 1, "Reconfigure called once");
}
```

#### Simulation Helper Pattern

Create `helper_simulate*()` functions that mirror contract logic:

```solidity
/// @notice Simulate absorption to calculate exact expected values
/// @dev Mirrors the contract's _absorbPremiumSteps logic exactly
function helper_simulateAbsorption(
    PackedSegment[] memory currentSegments_,
    uint oldFloorPrice_,
    uint oldFloorSupply_,
    uint collateralAmount_
) internal pure returns (AbsorptionResult memory result_) {
    result_.newFloorPrice = oldFloorPrice_;
    result_.newFloorSupply = oldFloorSupply_;

    uint remainingCollateral = collateralAmount_;

    // Mirror the contract's absorption loop exactly
    for (uint i = 1; i < currentSegments_.length && remainingCollateral > 0; i++) {
        (uint price, uint priceInc, uint supply, uint steps) =
            currentSegments_[i]._unpack();

        for (uint j = 0; j < steps && remainingCollateral > 0; j++) {
            uint stepPrice = price + (j * priceInc);
            uint stepCost = (stepPrice * supply) / 1e18;

            if (remainingCollateral >= stepCost) {
                remainingCollateral -= stepCost;
                result_.newFloorPrice = stepPrice;
                result_.newFloorSupply += supply;
                result_.lastConsumedSegmentIndex = i;
                result_.stepsConsumedFromLastSegment = j + 1;
            } else {
                break;
            }
        }
    }

    result_.collateralUsed = collateralAmount_ - remainingCollateral;
}
```

#### When Relative Assertions Are Acceptable

Only use relative assertions for truly non-deterministic values:

```solidity
// ACCEPTABLE: Gas usage is non-deterministic
uint gasBefore = gasleft();
_contract.doSomething();
uint gasUsed = gasBefore - gasleft();
assertLt(gasUsed, 100_000, "Should use less than 100k gas");

// ACCEPTABLE: Block timestamp with external calls that may advance time
// (rare, usually you control time in tests)

// ACCEPTABLE: When testing ranges explicitly (not a shortcut)
// e.g., "Fee must be between 1% and 5%"
assertGe(fee, minFee, "Fee meets minimum");
assertLe(fee, maxFee, "Fee under maximum");
// BUT: If you can calculate the exact fee, use assertEq instead!
```

### Phase 6: Run and Debug

#### Running Fuzz Tests

```bash
# Run all fuzz tests
forge test --match-test "testFuzz_"

# Run with more fuzz runs for deeper testing
forge test --match-test "testFuzz_" --fuzz-runs 1000

# Run specific test with verbose output
forge test --match-test "testFuzz_RaiseFloor_SingleStep" -vvvv
```

#### Foundry Configuration

In `foundry.toml`:

```toml
[fuzz]
runs = 256              # Default fuzz runs
max_test_rejects = 65536  # Max rejected inputs before failure
seed = '0x1'            # Optional: fixed seed for reproducibility

[profile.deep]
fuzz = { runs = 10000 }  # Deep fuzzing profile
```

#### Interpreting Failures

When a fuzz test fails, Foundry shows the failing inputs:

```
[FAIL. Reason: assertion failed]
        Counterexample: calldata=0x..., args=[1234, 5678, 0x...]
```

**Debug steps:**

1. **Reproduce with concrete test**:

   ```solidity
   function test_ReproduceFailure() public {
       // Use the exact values from the counterexample
       testFuzz_MyFunction(1234, 5678, address(0x...));
   }
   ```

2. **Add console logging**:

   ```solidity
   console.log("floorPrice_:", floorPrice_);
   console.log("calculated max:", maxCollateral);
   console.log("expected:", exp.newFloorPrice);
   ```

3. **Check simulation vs contract**:

   - If simulation and contract disagree, one has a bug
   - Usually the simulation helper needs fixing

4. **Verify bounds are correct**:
   - Was the input properly bounded?
   - Did derived constraints work correctly?

#### Coverage Verification

```bash
# Generate coverage report
forge coverage --match-test "testFuzz_" --report lcov

# View summary
forge coverage --match-test "testFuzz_"
```

## Common Patterns

### Pattern A: Pure Calculation Fuzz Test

```solidity
function testFuzz_CalculateFee(uint amount_, uint feeBps_) public {
    amount_ = bound(amount_, 1e18, 1e27);
    feeBps_ = bound(feeBps_, 1, 10_000);

    // Pre-calculate expected
    uint expectedFee = (amount_ * feeBps_) / 10_000;

    // Execute
    uint actualFee = _contract.calculateFee(amount_, feeBps_);

    // Exact verification
    assertEq(actualFee, expectedFee, "Fee matches expected");
}
```

### Pattern B: State Transition Fuzz Test

```solidity
function testFuzz_Deposit(address user_, uint amount_) public {
    vm.assume(user_ != address(0));
    vm.assume(user_.code.length == 0);
    amount_ = bound(amount_, 1e18, 1e27);

    // Setup
    _token.mint(user_, amount_);
    vm.prank(user_);
    _token.approve(address(_contract), amount_);

    // Pre-state
    uint balanceBefore = _contract.balanceOf(user_);
    uint totalBefore = _contract.totalDeposits();

    // Execute
    vm.prank(user_);
    _contract.deposit(amount_);

    // Exact verification
    assertEq(_contract.balanceOf(user_), balanceBefore + amount_);
    assertEq(_contract.totalDeposits(), totalBefore + amount_);
}
```

### Pattern C: Multi-Step Fuzz Test with Simulation

```solidity
function testFuzz_MultiStepOperation(
    uint param1_,
    uint param2_,
    uint seed_
) public {
    // Bound base parameters
    param1_ = bound(param1_, MIN, MAX);
    param2_ = bound(param2_, MIN2, MAX2);

    // Build fuzzed environment
    ComplexState memory state = helper_createFuzzedState(param1_, param2_, seed_);
    _contract.initialize(state);

    // Derive operation parameters from state
    uint operationAmount = helper_calculateSafeAmount(state);
    vm.assume(operationAmount > 0);

    // Simulate expected outcome
    ExpectedResult memory exp = helper_simulateOperation(state, operationAmount);

    // Execute
    _contract.performOperation(operationAmount);

    // Exact verification against simulation
    assertEq(_contract.getValue(), exp.expectedValue);
    assertEq(_contract.getState(), exp.expectedState);
    assertEq(_contract.getCounter(), exp.expectedCounter);
}
```

### Pattern D: Failure Case Fuzz Test

```solidity
function testFuzz_Revert_InsufficientBalance(
    address user_,
    uint balance_,
    uint withdrawAmount_
) public {
    vm.assume(user_ != address(0));
    balance_ = bound(balance_, 0, 1e27);
    withdrawAmount_ = bound(withdrawAmount_, balance_ + 1, type(uint128).max);

    // Setup: user has less than withdraw amount
    _setupUserBalance(user_, balance_);

    // Execute and expect revert
    vm.expectRevert(IContract.InsufficientBalance.selector);
    vm.prank(user_);
    _contract.withdraw(withdrawAmount_);
}
```

## Domain Examples

### Bonding Curve Segments

```solidity
function testFuzz_BuyOnCurve(
    uint depositAmount_,
    uint segmentCount_,
    uint seed_
) public {
    depositAmount_ = bound(depositAmount_, 1e18, 1e24);
    segmentCount_ = bound(segmentCount_, 2, 10);

    // Create fuzzed curve
    PackedSegment[] memory segments = helper_createFuzzedCurve(
        segmentCount_, seed_
    );
    _curve.configure(segments);

    // Derive max deposit from curve capacity
    uint maxDeposit = helper_calculateMaxDeposit(segments);
    depositAmount_ = bound(depositAmount_, 1e18, maxDeposit);

    // Simulate expected tokens
    uint expectedTokens = helper_simulatePurchase(segments, depositAmount_);
    vm.assume(expectedTokens > 0);

    // Execute
    _setupDeposit(depositAmount_);
    uint actualTokens = _curve.buy(depositAmount_, 0);

    // Exact verification
    assertEq(actualTokens, expectedTokens, "Tokens match simulation");
}
```

### Loop Position

```solidity
function testFuzz_CreateLoopPosition(
    uint collateral_,
    uint8 loops_,
    uint seed_
) public {
    collateral_ = bound(collateral_, 1e18, 1e24);
    loops_ = uint8(bound(loops_, 1, 5));

    // Calculate expected values
    uint expectedBorrow = helper_calculateBorrowAmount(collateral_, loops_);
    uint expectedPosition = collateral_ + expectedBorrow;
    uint[] memory expectedLoanIds = helper_generateLoanIds(loops_, seed_);

    // Execute
    (uint positionId, uint actualBorrow) = _facility.createLoopPosition(
        collateral_, loops_
    );

    // Exact verification
    assertEq(actualBorrow, expectedBorrow, "Borrow amount exact");

    Position memory pos = _facility.getPosition(positionId);
    assertEq(pos.collateral, collateral_, "Collateral exact");
    assertEq(pos.loops, loops_, "Loops exact");
    assertEq(pos.totalValue, expectedPosition, "Total value exact");
}
```

## Anti-Patterns (FORBIDDEN)

### Environment Creation Shortcuts

```solidity
// FORBIDDEN: Using hardcoded values
PackedSegment[] memory segments = _getDefaultSegments();  // Always same!

// FORBIDDEN: Skipping complex setup
// "Just test with simple case" - NO

// FORBIDDEN: Magic numbers
uint amount = 1000e18;  // Why 1000? Derive from state!

// FORBIDDEN: Arbitrary bounds
amount_ = bound(amount_, 1, 1000);  // Why 1000? Domain reason?

// FORBIDDEN: Same setup every run
function setUp() public {
    _createFixedTestState();  // Fuzz tests need fuzzed state!
}

// FORBIDDEN: Early returns for invalid combinations
if (a_ + b_ < c_) return;  // Lazy! Design bounds so this never happens

// FORBIDDEN: Helper functions that can fail
if (!helper_buildState(a_, b_)) return;  // Helper should always succeed!
```

### Verification Shortcuts

```solidity
// FORBIDDEN: Lazy greater-than
assertTrue(result > 0);  // What's the exact value?

// FORBIDDEN: Vague "increased"
assertTrue(newValue >= oldValue);  // By how much?

// FORBIDDEN: Range when exact is possible
assertGe(fee, 100);
assertLe(fee, 200);  // What's the exact fee?

// FORBIDDEN: Missing simulation
// Just check "it didn't revert" - NO, verify exact state!

// FORBIDDEN: Trusting the contract
assertEq(result, _contract.calculateExpected(...));
// ^ This just checks contract is internally consistent, not correct!
// Use an independent simulation helper instead.
```

## Checklist

### Before Writing Fuzz Test

- [ ] Identified function as good fuzz candidate
- [ ] **Identified correct test file section** (internal → internal section, external → external section)
- [ ] **Documented the property being tested** (Given/When/Then)
- [ ] Property statement is specific and verifiable
- [ ] Analyzed all input parameters and their domains
- [ ] Identified constraints between inputs
- [ ] Planned environment builder helpers needed
- [ ] Planned simulation helpers needed
- [ ] Decided: isolated (mocked internals) vs integration test?

### Environment Setup (NO SHORTCUTS)

- [ ] All fuzzable state is fuzzed (not hardcoded)
- [ ] Using `helper_createFuzzed*()` for complex state
- [ ] Bounds derived from fuzzed state (not magic numbers)
- [ ] Seed-based determinism for reproducibility
- [ ] **No early returns** - constraints designed so all combinations are valid
- [ ] Helper functions always succeed for bounded inputs
- [ ] Interdependent inputs derived (not independently bounded)
- [ ] If isolated test: mock switches configured, fuzzed return values set

### Verification (NO SHORTCUTS)

- [ ] Simulation helper exists and mirrors contract logic
- [ ] Pre-calculate ALL expected values before execution
- [ ] Using `assertEq` for ALL deterministic values
- [ ] No lazy `assertTrue(x >= y)` when exact is known
- [ ] No `assertGt(x, 0)` when exact value is calculable
- [ ] Events verified with exact expected parameters

### After Writing

- [ ] Test passes with default fuzz runs
- [ ] Test passes with increased fuzz runs (1000+)
- [ ] Coverage meets targets
- [ ] No TODO comments left unresolved
- [ ] Helper functions documented

## Key Principles Summary

1. **Identify**: Choose functions that benefit from input variety
2. **Bound properly**: Realistic ranges, domain-specific limits
3. **Fuzz the environment**: No hardcoded state—use fuzzed helpers
4. **Derive constraints**: Calculate bounds from fuzzed state
5. **No early returns**: Design constraints so all combinations are valid
6. **Isolate when needed**: Mock internal dependencies to test orchestration
7. **Simulate exactly**: Build helpers that mirror contract logic
8. **Assert exactly**: `assertEq` for all deterministic values
9. **NO SHORTCUTS**: Both environment AND verification require full rigor

**The Core Rules:**

1. **Environment**: If state can be fuzzed, fuzz it. If bounds can be derived, derive them. If you're early returning, your bounds are wrong.
2. **Verification**: If the math is deterministic, the assertion MUST be exact.
