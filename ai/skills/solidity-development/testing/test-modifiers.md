# Modifier Testing - Critical Rules

## Scope

**This rule ONLY covers modifier testing.** Do NOT implement functional tests (success cases, business logic, etc.) when applying this rule. Only implement:

1. Isolation tests for custom modifiers
2. In-position tests verifying modifiers are applied to functions

## TWO TYPES OF TESTS - COMPLETELY DIFFERENT

### Type 1: ISOLATION Tests (Test Modifier Logic)

- **Location:** "Test Modifier" section
- **What:** Test custom modifier logic in isolation
- **How:** Via `exposed_modifierName()` functions
- **Testing:** **USE FUZZING** - test edge cases, boundaries, all ranges
- **Goal:** Prove the modifier's validation logic works correctly

### Type 2: IN POSITION Tests (Verify Modifiers Applied)

- **Location:** "Test External Functions" section
- **What:** Verify modifiers are actually applied to functions
- **How:** Call actual function with bad inputs
- **Testing:** **CONCRETE VALUES ONLY** - no fuzzing
- **Gherkin:** MUST include "(modifier in position check)" in Then clauses
- **Goal:** Prove the modifier is on the function

## Prerequisites

Test skeleton must exist with:

- "Test Modifier" section with `/* Test: modifierName */` placeholders
- "Test External Functions" section with `/* Test: functionName */` placeholders
- Exposed mock contract created

## Critical Rule

**EVERY function with ANY modifiers needs a ModifierInPositionChecks test.**

This includes functions with ONLY `permissioned` modifier.

## Workflow

### Phase 1: Documentation First

#### Step 1: Analyze Contract

Read source contract and list:

- All custom modifiers (e.g., `loanIdIsValid`, `loopCountIsValid`)
- All external functions and their modifiers
- Which functions use which modifiers

#### Step 2: Write Isolation Gherkin

For EACH custom modifier in "Test Modifier" section:

```solidity
/*
Test: customModifier
├── Given: [Invalid condition - describe what makes it fail]
│   └── When: function using customModifier is called
│       └── Then: It should revert with [ErrorName]
└── Given: [Valid condition - describe what makes it pass]
    └── When: function using customModifier is called
        └── Then: It should not revert
*/
```

#### Step 3: Write In Position Gherkin

For EVERY function with modifiers in "Test External Functions" section:

**Single modifier:**

```solidity
/*
Test: functionName
└── Given: [Modifier condition fails]
    └── When: functionName is called
        └── Then: It should revert (modifier in position check)
*/
```

**Multiple modifiers:**

```solidity
/*
Test: functionName
├── Given: [Modifier 1 fails]
│   └── When: functionName is called
│       └── Then: It should revert (modifier in position check)
├── Given: [Modifier 2 fails]
│   └── When: functionName is called
│       └── Then: It should revert (modifier in position check)
└── Given: [Modifier 3 fails]
    └── When: functionName is called
        └── Then: It should revert (modifier in position check)
*/
```

**CRITICAL: Add "(modifier in position check)" to EVERY Then clause.**

### Phase 2: Implementation

#### Step 4: Add Exposed Functions (Custom Modifiers Only)

In mock contract:

```solidity
function exposed_customModifier(/* params */) external customModifier {}
```

Do NOT add exposed functions for `permissioned` or other standard modifiers.

#### Step 5: Implement Isolation Tests (WITH FUZZING)

**For parameter-based modifiers - USE FUZZING:**

```solidity
/* Test: loopCountIsValid */

function testLoopCountIsValid_FailsForInvalidValues(uint loops_) public {
    // Use vm.assume or bound to filter to invalid values
    vm.assume(loops_ == 0 || loops_ > _maxLoops);

    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__Error.selector
        )
    );
    _contract.exposed_loopCountIsValid(loops_);
}

function testLoopCountIsValid_WorksForValidValues(uint loops_) public {
    // Use bound to filter to valid range
    loops_ = bound(loops_, 1, _maxLoops);

    _contract.exposed_loopCountIsValid(loops_);
}
```

**For state-based modifiers - use concrete values:**

```solidity
/* Test: buyingIsEnabled */

function testBuyingIsEnabled_FailsIfNotEnabled() public {
    // State is disabled by default
    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__Error.selector
        )
    );
    _contract.exposed_buyingIsEnabled();
}

function testBuyingIsEnabled_WorksIfEnabled() public {
    _contract.enableBuy(); // Enable the state
    _contract.exposed_buyingIsEnabled();
}
```

**Goal: Test the modifier's logic thoroughly.**

#### Step 6: Implement In Position Tests (CONCRETE VALUES ONLY)

**For EACH function with modifiers:**

```solidity
/* Test: functionName */

function testFunctionName_ModifierInPositionChecks() public {
    // Test modifier 1
    // Setup condition that fails modifier 1
    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__Error1.selector
        )
    );
    _contract.functionName(/* concrete values */);

    // Reset/fix condition 1

    // Test modifier 2
    vm.expectRevert(
        abi.encodeWithSelector(
            IContract.Module__Error2.selector
        )
    );
    _contract.functionName(/* concrete values that fail modifier 2 */);

    // Continue for all modifiers...
}
```

**Example with permissioned:**

```solidity
/* Test: setMaxLoops */

function testSetMaxLoops_ModifierInPositionChecks() public {
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _creditFacility.setMaxLoops(10);
}
```

**Example with multiple modifiers:**

```solidity
/* Test: transferLoan */

function testTransferLoan_ModifierInPositionChecks() public {
    // Modifier 1: permissioned
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _creditFacility.transferLoan(1, address(0xBEEF));

    _authorizer.setAllAuthorized(true); // Reset

    // Modifier 2: onlyValidLoanId
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidLoanId.selector
        )
    );
    _creditFacility.transferLoan(0, address(0xBEEF));

    // More modifiers if exist...
}
```

**NO FUZZING in these tests. Use concrete values like 0, 1, address(0xB0B), etc.**

**Goal: Verify modifiers are on the function.**

### Phase 3: Verification

#### Step 7: Run Tests

```bash
forge test --mc ContractName_Test
```

Example:

```bash
forge test --mc Credit_Facility_v1_Test
```

Verify all tests pass.

## Complete Example

**Contract:**

```solidity
// Custom modifiers
modifier onlyValidLoanId(uint loanId) {
    if (!_loans[loanId].isActive || _loans[loanId].borrower != _msgSender()) {
        revert Module__Credit_Facility_InvalidLoanId();
    }
    _;
}

modifier loopCountIsValid(uint loops_) {
    _ensureLoopCountIsValid(loops_);
    _;
}

// Functions
function setMaxLoops(uint loops) external permissioned { }

function repay(uint loanId, uint amount)
    external
    permissioned
    loanIdIsValid(loanId)
    validAmount(amount)
{ }

function buyAndBorrow(uint amount, uint loops)
    external
    permissioned
    validAmount(amount)
    loopCountIsValid(loops)
{ }
```

**Exposed Mock:**

```solidity
function exposed_loanIdIsValid(uint loanId) external loanIdIsValid(loanId) {}
function exposed_loopCountIsValid(uint loops) external loopCountIsValid(loops) {}
```

**Test File:**

```solidity
///////////////////////////////////////////////////////////////////////////
// Test Modifier

/*
Test: onlyValidLoanId
├── Given: LoanId is invalid (not active or not owned by caller)
│   └── When: function using onlyValidLoanId is called
│       └── Then: It should revert
└── Given: LoanId is valid (active and owned by caller)
    └── When: function using onlyValidLoanId is called
        └── Then: It should not revert
*/

function testOnlyValidLoanId_FailsForInvalidLoanIds(uint loanId_) public {
    // FUZZING - test many invalid values
    vm.assume(loanId_ == 0 || loanId_ >= 100); // Assume no loans created

    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidLoanId.selector
        )
    );
    _creditFacility.exposed_onlyValidLoanId(loanId_);
}

function testOnlyValidLoanId_WorksForValidLoanId() public {
    // Create valid loan first
    uint loanId = _createTestLoan();
    _creditFacility.exposed_onlyValidLoanId(loanId);
}

/*
Test: loopCountIsValid
├── Given: Loops is zero or exceeds max
│   └── When: function using loopCountIsValid is called
│       └── Then: It should revert
└── Given: Loops is valid (1 to max)
    └── When: function using loopCountIsValid is called
        └── Then: It should not revert
*/

function testLoopCountIsValid_FailsForInvalidValues(uint loops_) public {
    // FUZZING - test range of invalid values
    vm.assume(loops_ == 0 || loops_ > _INITIAL_MAX_LOOPS);

    vm.expectRevert(
        abi.encodeWithSelector(
            ICreditFacility_v1.Module__CreditFacility__InvalidLoopCount.selector
        )
    );
    _creditFacility.exposed_loopCountIsValid(loops_);
}

function testLoopCountIsValid_WorksForValidValues(uint loops_) public {
    // FUZZING - test range of valid values
    loops_ = bound(loops_, 1, _INITIAL_MAX_LOOPS);
    _creditFacility.exposed_loopCountIsValid(loops_);
}

///////////////////////////////////////////////////////////////////////////
// Test External Functions

//--------------------------------------------------------------------------
// Administrative Functions

/*
Test: setMaxLoops
└── Given: Caller is not permissioned
    └── When: setMaxLoops is called
        └── Then: It should revert (modifier in position check)
*/

function testSetMaxLoops_ModifierInPositionChecks() public {
    // CONCRETE VALUES - just verify modifier is applied
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _creditFacility.setMaxLoops(10);
}

//--------------------------------------------------------------------------
// Loan Management

/*
Test: repay
├── Given: Caller is not permissioned
│   └── When: repay is called
│       └── Then: It should revert (modifier in position check)
├── Given: LoanId is invalid
│   └── When: repay is called
│       └── Then: It should revert (modifier in position check)
└── Given: Amount is zero
    └── When: repay is called
        └── Then: It should revert (modifier in position check)
*/

function testRepay_ModifierInPositionChecks() public {
    // CONCRETE VALUES ONLY - no fuzzing

    // Modifier 1: permissioned
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _creditFacility.repay(1, 1000e18);

    _authorizer.setAllAuthorized(true);

    // Modifier 2: onlyValidLoanId
    vm.expectRevert(
        abi.encodeWithSelector(
            ICredit_Facility_v1.Module__Credit_Facility_InvalidLoanId.selector
        )
    );
    _creditFacility.repay(0, 1000e18);

    // Modifier 3: validAmount
    uint loanId = _createTestLoan();
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__InvalidAmount.selector
        )
    );
    _creditFacility.repay(loanId, 0);
}

//--------------------------------------------------------------------------
// Borrowing

/*
Test: buyAndBorrow
├── Given: Caller is not permissioned
│   └── When: buyAndBorrow is called
│       └── Then: It should revert (modifier in position check)
├── Given: Amount is zero
│   └── When: buyAndBorrow is called
│       └── Then: It should revert (modifier in position check)
└── Given: Loop count is invalid
    └── When: buyAndBorrow is called
        └── Then: It should revert (modifier in position check)
*/

function testBuyAndBorrow_ModifierInPositionChecks() public {
    // CONCRETE VALUES

    // Modifier 1: permissioned
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _creditFacility.buyAndBorrow(1000e18, 2, false);

    _authorizer.setAllAuthorized(true);

    // Modifier 2: validAmount
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__InvalidAmount.selector
        )
    );
    _creditFacility.buyAndBorrow(0, 2, false);

    // Modifier 3: loopCountIsValid
    vm.expectRevert(
        abi.encodeWithSelector(
            ICreditFacility_v1.Module__CreditFacility__InvalidLoopCount.selector
        )
    );
    _creditFacility.buyAndBorrow(1000e18, 0, false);
}
```

## Testing `nonReentrant` Modifier

The `nonReentrant` modifier from OpenZeppelin's `ReentrancyGuardUpgradeable` requires special handling since it doesn't revert on invalid input - it prevents reentrant calls by writing to a storage slot.

### Helper Function (Already Implemented)

A helper function `_assertNonReentrantInPosition()` is available in `test/unit/core/modules/ModuleTest.sol`:

```solidity
/// @notice Asserts that the nonReentrant modifier was applied during a call.
/// @dev    Call vm.record() before the function under test, then call this
///         helper to verify the ReentrancyGuard storage slot was written to.
///         Note: vm.accesses() retrieves recorded data; call vm.record()
///         again if you need to start a new recording session.
/// @param  target_ The contract address to check storage writes for.
function _assertNonReentrantInPosition(address target_) internal {
    // OpenZeppelin ReentrancyGuardUpgradeable storage slot:
    // keccak256(abi.encode(uint256(keccak256("openzeppelin.storage.ReentrancyGuard")) - 1)) & ~bytes32(uint256(0xff))
    bytes32 reentrancySlot =
        0x9b779b17422d0df92223018b32b4d1fa46e071723d6817e2486d003becc55f00;

    (, bytes32[] memory writes) = vm.accesses(target_);

    for (uint i; i < writes.length; ++i) {
        if (writes[i] == reentrancySlot) return;
    }
    revert("nonReentrant modifier not applied");
}
```

### Usage in ModifierInPositionChecks Tests

```solidity
/*
Test: stake (modifiers)
├── Given: Caller is not permissioned
│   └── When: stake is called
│       └── Then: It should revert (modifier in position check)
├── Given: nonReentrant guard is not entered
│   └── When: stake is called
│       └── Then: It should write to reentrancy slot (modifier in position check)
└── ...
*/

function testStake_ModifierInPositionChecks() public {
    // Modifier 1: permissioned
    _authorizer.setAllAuthorized(false);
    vm.expectRevert(
        abi.encodeWithSelector(
            IModule_v2.Module__CallerNotPermissioned.selector
        )
    );
    vm.prank(address(0xB0B));
    _contract.stake(strategy, 1000e18);

    _authorizer.setAllAuthorized(true);

    // Modifier 2: nonReentrant (via storage slot check)
    // Setup: ensure function can execute successfully
    _setupForSuccessfulStake();
    vm.record();
    _contract.stake(strategy, 1000e18);
    _assertNonReentrantInPosition(address(_contract));

    // Continue with other modifiers...
}
```

### Key Points

1. **No isolation test needed** - `nonReentrant` is a standard OpenZeppelin modifier, not a custom one
2. **Test at the end of ModifierInPositionChecks** - Since the function must execute successfully, test `nonReentrant` last when all other modifiers have been satisfied and the environment is fully set up
3. **Use `vm.record()` before the call** - This starts recording storage accesses
4. **Call `_assertNonReentrantInPosition()` after** - This verifies the reentrancy slot was written to
5. **Function must execute successfully** - Unlike other modifiers that revert on bad input, we need the function to complete to verify `nonReentrant` was applied

## Summary - Critical Distinctions

| Aspect              | Isolation Tests          | In Position Tests                 |
| ------------------- | ------------------------ | --------------------------------- |
| **Location**        | "Test Modifier" section  | "Test External Functions" section |
| **What**            | Test modifier logic      | Verify modifier applied           |
| **Via**             | `exposed_modifierName()` | Actual function call              |
| **Which Modifiers** | Custom only              | ALL (including `permissioned`)    |
| **Testing Method**  | **FUZZING**              | **CONCRETE VALUES**               |
| **Gherkin Suffix**  | None                     | "(modifier in position check)"    |
| **Goal**            | Prove logic works        | Prove modifier is on function     |

**This rule ONLY covers modifier testing. Do NOT implement functional tests (success cases, business logic, etc.).**

## Checklist

- [ ] Phase 1: Write ALL Gherkin (isolation + in position)
- [ ] Phase 2: Add exposed functions to mock (custom modifiers only)
- [ ] Phase 2: Implement isolation tests WITH FUZZING
- [ ] Phase 2: Implement in position tests WITH CONCRETE VALUES
- [ ] Phase 2: Add "(modifier in position check)" to all in position Gherkin
- [ ] Phase 3: Run `forge test --mc ContractName_Test`
- [ ] Verify: Every custom modifier has isolation tests
- [ ] Verify: Every function with ANY modifier has ModifierInPositionChecks test
- [ ] Verify: NO functional tests added (only modifier tests)
