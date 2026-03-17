# External Function Testing Workflow

## Overview

External and public function tests are **integration tests** that verify the public API works correctly. This comes AFTER modifiers and internal functions are already tested.

**Key Focus:**

- Integration: Does the function call the correct internal functions?
- Public API: State changes, events, return values
- Don't re-test: Modifiers (done in test-modifiers.mdc) or internal logic (done in test-internal-functions.mdc)

**Testing Order:**

1. Modifiers tested → test-modifiers.mdc ✓
2. Internal functions tested → test-internal-functions.mdc ✓
3. External functions tested → **This guide** ← You are here

**Failure cases first → Success cases last**

## Prerequisites

- Test skeleton exists with "Test External Functions" section
- Subsections defined: "Public Functions - Getter Functions" and "Public Functions - Mutating Functions" (by category)
- ModifierInPositionChecks tests exist for each function with modifiers
- Internal functions already tested
- Modifiers already tested

## CRITICAL: Respect Structure and Avoid Duplication

**DO NOT test again:**

- ❌ Modifiers (already tested in test-modifiers.mdc via ModifierInPositionChecks)
- ❌ Internal function logic deeply (already tested in test-internal-functions.mdc)

**DO test:**

- ✓ Business logic failures (non-modifier)
- ✓ Integration (does it call the right internal functions?)
- ✓ State changes from the public API perspective
- ✓ Events emitted
- ✓ Return values
- ✓ Edge cases specific to the public API

**Respect section structure:**

- Keep tests in their existing subsections
- Getters stay in "Getter Functions"
- Mutating stay in their categories (Borrowing, Loan Management, Administrative, etc.)

## High-Level Workflow

### Phase 1: Documentation (Gherkin First)

Write ALL Gherkin before implementing any tests.

#### Step 1: Analyze Each Function

For each external/public function, identify:

- **Internal calls**: What internal functions does it call?
- **Business failures**: What non-modifier failures can occur?
- **State changes**: What gets modified?
- **Events**: What events are emitted?
- **Returns**: What does it return?
- **Edge cases**: Boundaries, zero values, max values?

#### Step 2: Write Gherkin (Failures First)

In existing subsection, expand `/* Test: functionName */` placeholder.

**Skip modifier conditions** - they're already documented in ModifierInPositionChecks tests.

**Pattern A: Getter (Trivial)**

```solidity
/* Test: getterFunction */
// Trivial
```

**Pattern B: Getter with Logic**

```solidity
/*
Test: calculateSomething
├── Given: Input parameter is invalid
│   └── When: calculateSomething is called
│       └── Then: It should revert with [ErrorName]
└── Given: Input parameter is valid
    └── When: calculateSomething is called
        └── Then: It should return [calculated value]
*/
```

**Pattern C: Mutating Function**

```solidity
/*
Test: mutatingFunction
├── Given: [Business rule violation - NOT modifier]
│   └── When: mutatingFunction is called
│       └── Then: It should revert with [ErrorName]
├── Given: [Another business rule violation]
│   └── When: mutatingFunction is called
│       └── Then: It should revert with [ErrorName]
└── Given: [All conditions valid]
    └── When: mutatingFunction is called
        └── Then: It should call [_internalFunction]
        └── And: It should update [state variable]
        └── And: It should emit [EventName]
        └── And: It should return [value]
*/
```

**Key:** Focus on business logic, integration, and API behavior.

#### Step 3: Review Documentation

- [ ] Every external/public function has Gherkin
- [ ] Skipped modifier conditions (already tested)
- [ ] Documented non-modifier failures
- [ ] Documented success behaviors
- [ ] Each in correct subsection

### Phase 2: Implementation

#### Step 4: Implement Tests

For each function, write tests matching Gherkin **in same subsection**.

**Naming:**

- `testFunctionName_FailsIfCondition()` - for failures
- `testFunctionName_SuccessCondition()` - for success

**Pattern A: Trivial Getter (No Test Needed)**

```solidity
/* Test: getMaxLoops */
// Trivial
```

**Pattern B: Getter with Logic (FUZZ SUCCESS CASES)**

```solidity
/* Test: calculatePurchaseReturn */

function testCalculatePurchaseReturn_FailsIfDepositZero() public {
    vm.expectRevert(
        IContract.Module__Error.selector
    );
    _contract.calculatePurchaseReturn(0);
}

function testCalculatePurchaseReturn_ReturnsCorrectAmount(uint deposit_) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    deposit_ = bound(deposit_, 1, type(uint128).max);

    uint result = _contract.calculatePurchaseReturn(deposit_);

    // Verify calculation relationship (not just > 0)
    uint expectedFee = (deposit_ * _contract.getFee()) / BPS;
    uint expectedReturn = deposit_ - expectedFee;
    assertEq(result, expectedReturn);
}
```

**Pattern C: Mutating Function (Failures → Success, FUZZ SUCCESS CASES)**

```solidity
/* Test: borrow */

function testBorrow_FailsIfInsufficientCollateral() public {
    // Setup: User has collateral but not enough
    _setupUser(address(0xBEEF), 100e18);

    vm.expectRevert(
        IContract.Module__InsufficientCollateral.selector
    );
    vm.prank(address(0xBEEF));
    _contract.borrow(10000e18); // Way too much
}

function testBorrow_CreatesLoanSuccessfully(
    address user_,
    uint collateral_,
    uint borrowAmount_
) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    // Bound to valid ranges
    vm.assume(user_ != address(0) && user_.code.length == 0);
    collateral_ = bound(collateral_, 1000e18, type(uint128).max);
    borrowAmount_ = bound(borrowAmount_, 1e18, collateral_ / 2); // Ensure sufficient collateral

    // Setup with fuzzed values
    _setupUserWithCollateral(user_, collateral_);

    // Expect event with fuzzed params (shows internal was called and completed)
    vm.expectEmit(true, true, true, true);
    emit LoanCreated(1, user_, borrowAmount_, collateral_);

    // Call with fuzzed params
    vm.prank(user_);
    uint loanId = _contract.borrow(borrowAmount_);

    // Verify integration: internal function effects
    assertEq(loanId, 1);

    // Verify state changes match fuzzed inputs
    (address borrower, uint amount,,, bool active) = _contract.getLoan(loanId);
    assertEq(borrower, user_);
    assertEq(amount, borrowAmount_);
    assertTrue(active);

    // Verify user's loan array updated
    uint[] memory userLoans = _contract.getUserLoanIds(user_);
    assertEq(userLoans.length, 1);
    assertEq(userLoans[0], loanId);
}
```

**Pattern D: Verify Internal Function Called**

You can't directly test which internal function was called, but you can verify its effects:

```solidity
function testFunction_CallsInternalCorrectly() public {
    // Setup

    // Call external function
    _contract.externalFunction(params);

    // Verify state changes that ONLY happen if the internal was called
    // e.g., if internal increments a counter, check counter
    // if internal emits event, check event
    // if internal modifies specific state, check that state
}
```

**Fuzzing Guidelines (FUZZ SUCCESS CASES - PREFERRED):**

- Getters with parameters: Fuzz the inputs for success cases
- Mutating functions: Fuzz valid inputs for success cases, make helpers support fuzzed setup
- Complex integration: Fuzz when possible, use helpers for complex state setup
- Use concrete values only when fuzzing setup is impractical
- Verify mathematical relationships and ranges, not just exact values

### Phase 3: Verification & Cleanup

#### Step 5: Run Tests with Coverage

**Run tests:**

```bash
forge test --mc ContractName_Test
```

**Generate coverage and update lcov file:**

```bash
forge coverage --mc ContractName_Test --report lcov
```

This updates the `lcov.info` file which is used for coverage analysis.

**Check coverage:**

```bash
forge coverage --mc ContractName_Test
```

**Coverage Checklist:**

- [ ] All external functions have tests
- [ ] All public functions have tests
- [ ] Coverage > 90% for the contract
- [ ] No untested branches in external/public functions
- [ ] If coverage < 90%, identify missing tests

#### Step 6: Cleanup TODOs

**Find TODOs:**

```bash
grep -n "// TODO" test/unit/path/ContractName_Test.sol
```

**For each TODO found:**

1. **Can it be implemented now?**
   - If yes: Implement it
   - If no: Continue to step 2

2. **Needs more information?**
   - Document what's needed
   - Ask user for clarification
   - Continue to step 3

3. **Is it tech debt?**
   - Create a GitHub issue
   - Link issue number in comment
   - Update comment: `// TODO(#123): Description`

4. **Remove if completed**
   - Delete `// TODO:` comments for tests that are now implemented

**TODO Cleanup Checklist:**

- [ ] Listed all TODOs found in test file
- [ ] Categorized each: implement now / needs info / tech debt
- [ ] Implemented what can be done now
- [ ] Created issues for tech debt
- [ ] Updated TODO comments with issue numbers
- [ ] Removed completed TODOs
- [ ] Presented remaining TODOs to user

## Complete Example

Based on IssuanceBase_v2.t.sol patterns:

```solidity
///////////////////////////////////////////////////////////////////////////
// Test External Functions

//===========================================================================
// Public Functions - Getter Functions

/* Test: getBuyFee */
// Trivial

/* Test: calculatePurchaseReturn */

function testCalculatePurchaseReturn_FailsIfDepositZero() public {
    vm.expectRevert(
        IIssuanceBase_v2.Module__IssuanceBase__InvalidDepositAmount.selector
    );
    _issuanceBase.calculatePurchaseReturn(0);
}

function testCalculatePurchaseReturn_ReturnsCorrectAmount(
    uint depositAmount_,
    uint feeAmount_
) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    depositAmount_ = bound(depositAmount_, 1, type(uint128).max);
    uint bps = _issuanceBase.BPS();
    feeAmount_ = bound(feeAmount_, 1, bps);

    vm.assume(feeAmount_ * depositAmount_ > bps); // Fee not zero

    _issuanceBase.setBuyFee(feeAmount_);

    uint result = _issuanceBase.calculatePurchaseReturn(depositAmount_);

    // Verify mathematical relationship with fuzzed inputs
    uint expectedFee = (depositAmount_ * feeAmount_) / bps;
    uint expectedNet = depositAmount_ - expectedFee;

    assertEq(result, expectedNet);
}

//==========================================================================
// Public Functions - Mutating Functions

//--------------------------------------------------------------------------
// Buy Functions

/* Test: buy */

function testBuy_ForwardsToInternalBuyOrder(
    uint depositAmount_,
    uint minAmountOut_
) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    // Bound to valid ranges
    depositAmount_ = bound(depositAmount_, 1e18, type(uint128).max);
    minAmountOut_ = bound(minAmountOut_, 1, depositAmount_);

    // Setup with fuzzed amounts
    _issuanceBase.enableBuy();
    _collateralToken.mint(address(this), depositAmount_);
    _collateralToken.approve(address(_issuanceBase), depositAmount_);

    uint balanceBefore = _issuanceToken.balanceOf(address(this));

    // Calculate expected return with fuzzed input
    uint expectedReturn = _issuanceBase.calculatePurchaseReturn(depositAmount_);

    // Expect event with fuzzed params (proves internal _buyOrder was called)
    vm.expectEmit(true, true, true, true);
    emit IIssuanceBase_v2.TokensBought(
        address(this),
        depositAmount_,
        expectedReturn,
        address(this)
    );

    // Call with fuzzed params
    _issuanceBase.buy(depositAmount_, minAmountOut_);

    // Verify integration: tokens were minted (internal function effect)
    uint balanceAfter = _issuanceToken.balanceOf(address(this));
    assertEq(balanceAfter - balanceBefore, expectedReturn);
}

//--------------------------------------------------------------------------
// Administrative Functions

/* Test: setBuyFee */

function testSetBuyFee_FailsIfFeeInvalid() public {
    uint invalidFee = _issuanceBase.BPS() + 1;

    vm.expectRevert(
        IIssuanceBase_v2.Module__IssuanceBase__InvalidFeePercentage.selector
    );
    _issuanceBase.setBuyFee(invalidFee);
}

function testSetBuyFee_UpdatesFeeAndEmitsEvent(uint fee_) public {
    fee_ = bound(fee_, 1, _issuanceBase.BPS());

    uint oldFee = _issuanceBase.getBuyFee();

    vm.expectEmit(true, true, true, true);
    emit IIssuanceBase_v2.BuyFeeUpdated(fee_, oldFee);

    _issuanceBase.setBuyFee(fee_);

    assertEq(_issuanceBase.getBuyFee(), fee_);
}
```

## Key Principles

### 1. Integration Testing, Not Re-Testing

**You are testing:**

- Public API contracts
- Integration between public and internal functions
- End-to-end behavior from user perspective

**You are NOT testing:**

- Modifier logic (done in test-modifiers.mdc)
- Internal function logic deeply (done in test-internal-functions.mdc)

### 2. Verify Integration via Side Effects

Since you can't directly check "was internal function X called", verify:

- State changes that only happen if internal was called
- Events that internal functions emit
- Return values that come from internal functions
- Effects on other contracts (token transfers, etc.)

### 3. Failure Cases First

Test order:

1. Non-modifier business logic failures
2. Edge cases and boundaries
3. Success cases with state/event/return verification

### 4. Use Helper Functions

Complex setup? Create helpers in "Helper Functions" section:

```solidity
///////////////////////////////////////////////////////////////////////////
// Helper Functions

function _setupUserWithCollateral(address user, uint amount) internal {
    _collateralToken.mint(user, amount);
    vm.prank(user);
    _collateralToken.approve(address(_contract), amount);
}

function _createTestLoan() internal returns (uint loanId) {
    // Create and return a valid loan for testing
}
```

## Testing Strategy by Function Type

### Trivial Getters

- Mark as `// Trivial`
- No test needed (just returns storage)

### Getters with Logic

- Failure cases: Test invalid inputs → revert (concrete or fuzzed)
- Success cases: Fuzz valid inputs → verify mathematical relationships (PREFERRED)
- Verify calculations with relationship assertions

### Mutating Functions

- Failure cases: Test business logic failures (not modifiers) - concrete or fuzzed
- Success cases: Fuzz valid inputs for state changes, events, returns (PREFERRED)
- Verify integration with internal functions via effects
- Make helper functions support fuzzed setup values
- Use range/relationship assertions for state verification

## Checklist

**Phase 1 - Documentation:**

- [ ] Read contract, analyze all external/public functions
- [ ] Identify what internal functions each calls
- [ ] Write Gherkin for all (skip modifiers, failures first)
- [ ] Keep in existing subsections
- [ ] Review: every function documented

**Phase 2 - Implementation:**

- [ ] Implement failure tests (non-modifier failures - concrete or fuzzed)
- [ ] Implement success tests with fuzzing (state/events/returns) - PREFERRED
- [ ] Verify integration via side effects using relationship assertions
- [ ] Fuzz valid inputs for success cases when possible
- [ ] Create helper functions that support fuzzed inputs for complex setup
- [ ] Tests in same subsections as Gherkin

**Phase 3 - Verification & Cleanup:**

- [ ] Run `forge test --mc ContractName_Test` (all pass)
- [ ] Generate coverage with `forge coverage --mc ContractName_Test --report lcov`
- [ ] Check coverage with `forge coverage --mc ContractName_Test`
- [ ] Verify coverage > 90%
- [ ] Find all TODOs with `grep -n "// TODO"`
- [ ] Categorize TODOs: implement / needs info / tech debt
- [ ] Implement what can be done
- [ ] Create issues for tech debt
- [ ] Update TODO comments with issue numbers
- [ ] Remove completed TODOs
- [ ] Present remaining TODOs to user

## Final Reminder

**What makes external function tests different:**

- **Unit tests** (internal) = Test business logic in isolation
- **Integration tests** (external) = Test public API and integration

Focus on the API contract and integration, not re-testing internals.
