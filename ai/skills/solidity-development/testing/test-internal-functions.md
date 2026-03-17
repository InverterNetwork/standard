# Internal Function Testing Workflow

## Overview

Internal functions are tested via exposed functions in the mock contract. This guide focuses on testing business logic and state changes - **modifiers are already tested separately**.

**Testing Strategy:**

1. **Document** - Write Gherkin for all internal functions (failures first → success last)
2. **Implement** - Write test code matching the Gherkin
3. **Verify** - Run tests

## Prerequisites

- Test skeleton exists with "Test Internal Functions" section
- This section has subsections already defined (e.g., "Internal - Price & Calculation", "Internal - Borrowing", etc.)
- Each subsection has `/* Test: _internalFunction */` placeholders
- Exposed mock contract exists with `exposed_internalFunction()` wrappers
- Modifiers already tested (skip modifier logic in these tests)

## CRITICAL: Respect Existing Section Structure

**DO NOT move tests between sections.**

Tests must stay in their designated subsections:

- Tests in "Internal - Price & Calculation" stay there
- Tests in "Internal - Borrowing" stay there
- Tests in "Internal - Loan Management" stay there
- Tests in "Internal - Token Operations" stay there
- Tests in "Internal - State Management" stay there

**The skeleton already defines where each function belongs. Do not reorganize.**

## High-Level Workflow

### Phase 1: Documentation (Gherkin First)

Read the contract and document ALL internal functions before writing any test code.

#### Step 1: Analyze Internal Functions

For each internal function, identify:

- **Input validation** - What parameters can be invalid?
- **State requirements** - What state must exist before calling?
- **Business logic** - What calculations or operations happen?
- **State changes** - What gets modified?
- **Events emitted** - What events fire?
- **Return values** - What does it return?
- **Edge cases** - Boundaries, zero values, max values?

#### Step 2: Write Gherkin (Failures First)

In the "Test Internal Functions" section, expand each `/* Test: _internalFunction */` placeholder **in the subsection where it already exists**.

**IMPORTANT:** Find the function placeholder in the skeleton and write the Gherkin there. Do not move it to a different subsection.

**Standard Pattern (Failures → Success):**

```solidity
/*
Test: _internalFunction
├── Given: [Parameter 1 is invalid]
│   └── When: _internalFunction is called
│       └── Then: It should revert with [ErrorName]
├── Given: [Parameter 2 is invalid]
│   └── When: _internalFunction is called
│       └── Then: It should revert with [ErrorName]
├── Given: [Required state doesn't exist]
│   └── When: _internalFunction is called
│       └── Then: It should revert with [ErrorName]
├── Given: [Business rule violation]
│   └── When: _internalFunction is called
│       └── Then: It should revert with [ErrorName]
└── Given: [All conditions valid]
    └── When: _internalFunction is called
        └── Then: It should [perform operation]
        └── And: It should update [state variable]
        └── And: It should emit [EventName]
        └── And: It should return [expected value]
*/
```

**Key Principle:** Document failure cases first, success cases last.

#### Step 3: Review Documentation

Before implementing, verify:

- [ ] Every internal function has Gherkin
- [ ] All failure cases documented
- [ ] All success behaviors documented (state changes, events, returns)
- [ ] Each Gherkin is in the correct subsection (as defined in skeleton)
- [ ] No functions moved to different subsections

### Phase 2: Implementation (Tests After Docs)

#### Step 4: Implement Tests

For each internal function, create tests matching the Gherkin **in the same subsection**.

**CRITICAL:** Write tests directly below the Gherkin comment in the subsection where it exists. Do not move tests to different subsections.

**Naming Convention:**

- `test_InternalFunction_FailureCondition()` - for failure cases
- `test_InternalFunction_SuccessCondition()` - for success cases

**Pattern A: Input Validation (can use fuzzing):**

```solidity
/* Test: _internalFunction */

function test_InternalFunction_FailsIfParamInvalid(uint param_) public {
    // Filter to invalid values
    vm.assume(param_ == 0);

    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__Error.selector
        )
    );
    _contract.exposed_internalFunction(param_);
}
```

**Pattern B: State Requirements:**

```solidity
function test_InternalFunction_FailsIfStateNotReady() public {
    // Don't setup required state
    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__StateError.selector
        )
    );
    _contract.exposed_internalFunction(/* params */);
}
```

**Pattern C: Business Logic Validation:**

```solidity
function test_InternalFunction_FailsIfBusinessRuleViolated() public {
    // Setup state that violates business rule
    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__BusinessRuleError.selector
        )
    );
    _contract.exposed_internalFunction(/* params */);
}
```

**Pattern D: Success Path with Concrete Values (use when fuzzing isn't practical):**

```solidity
function test_InternalFunction_SuccessCondition() public {
    // Setup valid state

    // Expect event
    vm.expectEmit(true, true, true, true);
    emit EventName(/* expected params */);

    // Call function
    uint result = _contract.exposed_internalFunction(/* params */);

    // Assert state changes
    assertEq(_contract.stateVariable(), expectedValue);

    // Assert return value
    assertEq(result, expectedReturn);
}
```

**Pattern E: Fuzzed Success Path (PREFERRED for success cases):**

```solidity
function test_InternalFunction_WorksForValidRange(
    uint param1_,
    uint param2_,
    address user_
) public {
    // Bound to valid ranges
    param1_ = bound(param1_, MIN_VALUE, MAX_VALUE);
    param2_ = bound(param2_, 1, type(uint128).max);
    vm.assume(user_ != address(0));

    // Setup state if needed
    _setupValidState(param1_, param2_);

    // Expect event (with fuzzed params)
    vm.expectEmit(true, true, true, true);
    emit EventName(user_, param1_, param2_);

    // Call function
    uint result = _contract.exposed_internalFunction(param1_, param2_, user_);

    // Assert state changes (use relationships, not exact values)
    assertGt(_contract.stateVariable(), 0);

    // Assert return value (verify mathematical relationships)
    assertEq(result, (param1_ * param2_) / SCALE);
    // Or use range checks: assertGe(result, minExpected) && assertLe(result, maxExpected)
}
```

### Phase 3: Verification

#### Step 5: Run Tests and Generate Coverage

**Run tests:**

```bash
forge test --mc ContractName_Test
```

**Generate coverage and update lcov file:**

```bash
forge coverage --mc ContractName_Test --report lcov
```

This updates the `lcov.info` file which is used for coverage analysis.

Verify all internal function tests pass.

## Complete Example

**Source Contract:**

```solidity
function _calculateBorrowAmount(
    uint collateralAmount,
    uint loops,
    uint floorPrice
) internal pure returns (uint borrowAmount) {
    if (collateralAmount == 0) revert Module__InvalidAmount();
    if (loops == 0) revert Module__CreditFacility_InvalidLoopCount();
    if (floorPrice == 0) revert Module__InvalidPrice();

    borrowAmount = (collateralAmount * loops * floorPrice) / 1e18;

    if (borrowAmount == 0) revert Module__CalculationResultedInZero();

    return borrowAmount;
}

function _createLoan(
    address borrower,
    uint borrowAmount,
    uint collateralAmount,
    uint floorPrice
) internal returns (uint loanId) {
    if (borrower == address(0)) revert Module__InvalidAddress();
    if (borrowAmount == 0) revert Module__InvalidAmount();

    loanId = _nextLoanId++;

    _loans[loanId] = Loan({
        borrower: borrower,
        borrowAmount: borrowAmount,
        collateralAmount: collateralAmount,
        floorPrice: floorPrice,
        isActive: true
    });

    _userLoans[borrower].push(loanId);

    emit LoanCreated(loanId, borrower, borrowAmount, collateralAmount);

    return loanId;
}
```

**Exposed Mock:**

```solidity
function exposed_calculateBorrowAmount(
    uint collateralAmount,
    uint loops,
    uint floorPrice
) external pure returns (uint) {
    return _calculateBorrowAmount(collateralAmount, loops, floorPrice);
}

function exposed_createLoan(
    address borrower,
    uint borrowAmount,
    uint collateralAmount,
    uint floorPrice
) external returns (uint) {
    return _createLoan(borrower, borrowAmount, collateralAmount, floorPrice);
}
```

**Phase 1: Gherkin Documentation**

```solidity
///////////////////////////////////////////////////////////////////////////
// Test Internal Functions

//==========================================================================
// Internal Functions

//--------------------------------------------------------------------------
// Internal - Calculations

/*
Test: _calculateBorrowAmount
├── Given: Collateral amount is zero
│   └── When: _calculateBorrowAmount is called
│       └── Then: It should revert with Module__InvalidAmount
├── Given: Loops is zero
│   └── When: _calculateBorrowAmount is called
│       └── Then: It should revert with Module__CreditFacility_InvalidLoopCount
├── Given: Floor price is zero
│   └── When: _calculateBorrowAmount is called
│       └── Then: It should revert with Module__InvalidPrice
├── Given: Calculation results in zero
│   └── When: _calculateBorrowAmount is called
│       └── Then: It should revert with Module__CalculationResultedInZero
└── Given: All parameters valid
    └── When: _calculateBorrowAmount is called
        └── Then: It should return correct borrow amount
*/

//--------------------------------------------------------------------------
// Internal - State Management

/*
Test: _createLoan
├── Given: Borrower address is zero
│   └── When: _createLoan is called
│       └── Then: It should revert with Module__InvalidAddress
├── Given: Borrow amount is zero
│   └── When: _createLoan is called
│       └── Then: It should revert with Module__InvalidAmount
└── Given: All parameters valid
    └── When: _createLoan is called
        └── Then: It should create loan with correct data
        └── And: It should increment loan ID counter
        └── And: It should add loan to user's loan array
        └── And: It should emit LoanCreated event
        └── And: It should return the new loan ID
*/
```

**Phase 2: Implementation**

```solidity
///////////////////////////////////////////////////////////////////////////
// Test Internal Functions

//==========================================================================
// Internal Functions

//--------------------------------------------------------------------------
// Internal - Calculations

/* Test: _calculateBorrowAmount */

function test_CalculateBorrowAmount_FailsIfCollateralZero() public {
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidAmount.selector
        )
    );
    _creditFacility.exposed_calculateBorrowAmount(0, 2, 1e18);
}

function test_CalculateBorrowAmount_FailsIfLoopsZero() public {
    vm.expectRevert(
        abi.encodeWithSelector(
            ICreditFacility_v1.Module__CreditFacility_InvalidLoopCount.selector
        )
    );
    _creditFacility.exposed_calculateBorrowAmount(1000e18, 0, 1e18);
}

function test_CalculateBorrowAmount_FailsIfPriceZero() public {
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidPrice.selector
        )
    );
    _creditFacility.exposed_calculateBorrowAmount(1000e18, 2, 0);
}

function test_CalculateBorrowAmount_FailsIfResultZero() public {
    // Very small values that result in zero after division
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_CalculationResultedInZero.selector
        )
    );
    _creditFacility.exposed_calculateBorrowAmount(1, 1, 1);
}

function test_CalculateBorrowAmount_ReturnsCorrectAmount(
    uint collateral_,
    uint loops_,
    uint price_
) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    // Bound to reasonable ranges
    collateral_ = bound(collateral_, 1e18, 1000000e18);
    loops_ = bound(loops_, 1, 10);
    price_ = bound(price_, 0.01e18, 1000e18);

    uint result = _creditFacility.exposed_calculateBorrowAmount(
        collateral_,
        loops_,
        price_
    );

    // Verify mathematical relationship
    uint expected = (collateral_ * loops_ * price_) / 1e18;
    assertEq(result, expected);

    // Verify non-zero
    assertGt(result, 0);
}

//--------------------------------------------------------------------------
// Internal - State Management

/* Test: _createLoan */

function test_CreateLoan_FailsIfBorrowerZero() public {
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__InvalidAddress.selector
        )
    );
    _creditFacility.exposed_createLoan(address(0), 1000e18, 500e18, 1e18);
}

function test_CreateLoan_FailsIfAmountZero() public {
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidAmount.selector
        )
    );
    _creditFacility.exposed_createLoan(address(this), 0, 500e18, 1e18);
}

function test_CreateLoan_CreatesLoanCorrectly(
    address borrower_,
    uint borrowAmount_,
    uint collateralAmount_,
    uint floorPrice_
) public {
    // FUZZED SUCCESS CASE (PREFERRED)
    // Bound to valid ranges
    vm.assume(borrower_ != address(0) && borrower_.code.length == 0);
    borrowAmount_ = bound(borrowAmount_, 1e18, type(uint128).max);
    collateralAmount_ = bound(collateralAmount_, 1e18, type(uint128).max);
    floorPrice_ = bound(floorPrice_, 0.01e18, 10000e18);

    // Expect event with fuzzed params
    vm.expectEmit(true, true, true, true);
    emit LoanCreated(1, borrower_, borrowAmount_, collateralAmount_);

    // Create loan with fuzzed params
    vm.prank(borrower_);
    uint loanId = _creditFacility.exposed_createLoan(
        borrower_,
        borrowAmount_,
        collateralAmount_,
        floorPrice_
    );

    // Verify loan ID
    assertEq(loanId, 1);

    // Verify loan data matches fuzzed inputs
    (
        address storedBorrower,
        uint storedBorrowAmount,
        uint storedCollateral,
        uint storedPrice,
        bool isActive
    ) = _creditFacility.getLoan(loanId);

    assertEq(storedBorrower, borrower_);
    assertEq(storedBorrowAmount, borrowAmount_);
    assertEq(storedCollateral, collateralAmount_);
    assertEq(storedPrice, floorPrice_);
    assertTrue(isActive);

    // Verify loan added to user's array
    uint[] memory userLoans = _creditFacility.getUserLoanIds(borrower_);
    assertEq(userLoans.length, 1);
    assertEq(userLoans[0], loanId);
}
```

**Phase 3: Run Tests**

```bash
forge test --mc Credit_Facility_v1_Test
```

## Function Categories

Organize tests by category as in the skeleton:

```solidity
//--------------------------------------------------------------------------
// Internal - Price & Calculation

/* Test: _getFloorPrice */
/* Test: _calculateInterest */

//--------------------------------------------------------------------------
// Internal - Borrowing

/* Test: _borrow */
/* Test: _buyAndBorrow */

//--------------------------------------------------------------------------
// Internal - Loan Management

/* Test: _createLoan */
/* Test: _consolidateLoans */

//--------------------------------------------------------------------------
// Internal - Token Operations

/* Test: _executeTokenTransfer */

//--------------------------------------------------------------------------
// Internal - State Management

/* Test: _removeLoanFromUserLoans */
```

## Testing Strategy by Function Type

### Pure Functions (Calculations)

- Failure cases: Fuzz invalid inputs
- Success cases: Fuzz valid inputs (PREFERRED)
- Verify mathematical correctness with relationship assertions
- No state to setup or verify

### State-Modifying Functions

- Failure cases: Fuzz or use concrete invalid inputs
- Success cases: Fuzz valid inputs (PREFERRED)
- Setup required state (make helpers support fuzzed inputs)
- Verify ALL state changes (use relationship checks, not exact values)
- Check events emitted (with fuzzed params)
- Validate return values (verify mathematical relationships or ranges)

### Complex Business Logic

- Test all failure paths
- Test edge cases
- Verify state transitions
- Check side effects

## Key Principles

**1. Failure Cases First**

- Test all reverts before testing success
- Document why each revert happens
- Use specific error selectors

**2. Complete Success Testing with Fuzzing (PREFERRED)**

- Use fuzzing for success cases to test across input ranges
- Verify state changes (use relationships, not exact values)
- Check events (with fuzzed params)
- Validate returns (verify mathematical relationships or ranges)
- Use concrete values only when fuzzing isn't practical

**3. No Modifier Testing**

- Internal functions typically don't have modifiers
- Modifiers already tested in test-modifiers.mdc
- Focus on business logic and state changes

**4. Use Fuzzing for Success Cases (PREFERRED)**

- Failure cases: Fuzz invalid inputs to test validation
- Success cases: Fuzz valid inputs to test across ranges
- Pure calculations: Excellent for fuzzing with mathematical relationship assertions
- State changes: Fuzz inputs, verify state relationships
- Complex interactions: Fuzz when possible, use helpers for setup
- Use concrete values only when fuzzing setup is impractical

## Handling Complex Internal Functions

### RULE: "Too Complex" is NOT a Valid Excuse

**Never skip tests because:**

- "Complex setup required" → Build helper functions
- "Tested through public functions" → Not good enough
- "Requires mock behavior" → Then add the mock behavior

**Why implicit testing fails:**

- Doesn't test all code paths in the internal function
- Hides which step failed when tests break
- Missing edge cases
- False coverage metrics

### Solution: Build Test Infrastructure

**Helper Functions Pattern:**

```solidity
// In Helper Functions section
function _setupComplexDependency() internal {
    // Reusable setup logic
}

/* Test: _complexFunction */

function test_ComplexFunction_FailsIfCondition() public {
    _setupComplexDependency();

    vm.expectRevert(/* error */);
    _contract.exposed_complexFunction(/* params */);
}

function test_ComplexFunction_Success() public {
    _setupComplexDependency();

    uint result = _contract.exposed_complexFunction(/* params */);
    assertEq(result, expected);
}
```

**Key approach:** Create setup helpers, don't skip tests.

### If Truly Impossible (Very Rare)

Only skip if:

1. External non-deterministic dependencies that cannot be mocked
2. Would require weeks of infrastructure work
3. Simple enough for code review

**Must document:**

```solidity
/* Test: _functionName */

// SKIPPED: [Specific technical reason]
// Infrastructure needed: [Specific items]
// TODO: Issue #[number] created
//
// Behaviors that SHOULD be tested:
// - [List all cases that need testing]
```

**Then create a tech debt issue immediately.**

### Checklist Before Skipping

- [ ] Tried building helper functions?
- [ ] Mocked external dependencies?
- [ ] Estimated effort? (usually < 1 day)
- [ ] Created tech debt issue?
- [ ] Documented what should be tested?
- [ ] Got tech lead approval?

If any "no" → **keep working on the test.**

## Checklist

**Phase 1 - Documentation:**

- [ ] Analyze all internal functions
- [ ] Write Gherkin for all internal functions (failures first)
- [ ] Keep each function in its existing subsection from skeleton
- [ ] Do NOT move functions between subsections
- [ ] Review documentation completeness

**Phase 2 - Implementation:**

- [ ] Implement failure tests in the same subsection as Gherkin
- [ ] Implement success tests in the same subsection as Gherkin
- [ ] Do NOT reorganize or move tests between subsections
- [ ] Add fuzzing where appropriate
- [ ] Tests appear directly below their Gherkin comments

**Phase 3 - Verification:**

- [ ] Run `forge test --mc ContractName_Test`
- [ ] Generate coverage with `forge coverage --mc ContractName_Test --report lcov`
- [ ] Verify: Every internal function tested
- [ ] Verify: All failure paths covered
- [ ] Verify: All success behaviors verified
- [ ] Verify: Tests are in correct subsections (not moved)
