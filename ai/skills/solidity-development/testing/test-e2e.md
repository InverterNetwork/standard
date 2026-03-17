# E2E Test Writing Guide

## Overview

End-to-End (E2E) tests verify complete Floor workflows by testing multiple modules working together. Unlike unit tests that isolate components, E2E tests simulate real-world scenarios from a user's perspective.

**Key Focus:**

- Integration: Multiple modules interacting correctly
- Workflows: Complete user journeys (buy → sell, role assignment → function execution)
- Authorization: Role-based and token-gated access control
- Real scenarios: Demonstrating how the system works in practice

**E2E tests are documentation** - they show other developers how to use the system and what behaviors to expect.

## Test Architecture

### Contract Hierarchy

```
Test (Forge)
└── E2EModuleRegistry
    ├── Module setup helpers (setUpRoleAuthorizer, setUpBCDiscreteRedeemingVirtualSupply, etc.)
    └── E2ETest (abstract)
        ├── Common infrastructure (token, floorFactory, gov, etc.)
        ├── _configureSegments() [abstract - must implement]
        └── YourE2ETest
```

### File Organization

```
test/e2e/
├── E2ETest.sol                    # Base contract with common setup
├── E2EModuleRegistry.sol          # Module configurations and helpers
└── core/
    └── authorizer/
        ├── RoleAuthorizerE2E.t.sol           # Role-based access control
        └── TokenGatedRoleAuthorizerE2E.t.sol # Token-gated roles
```

## Core Concepts

### Floors

A **Floor** is the bonding curve contract that serves as the entry point. It coordinates:

- Issuance token minting/burning
- Collateral token management
- Access control via an Authorizer
- Fee collection via a Treasury

### Module Configuration Order

Modules MUST be configured in this exact order (enforced by factory):

```
[0] Floor Module       → The bonding curve (e.g., BC_Discrete_Redeeming_VirtualSupply_v1)
[1] Authorizer Module  → Access control (e.g., AUT_Roles_v2, AUT_TokenGated_Roles_v2)
[2] Treasury Module    → Fee collection (e.g., SplitterTreasury_v1)
[3+] Optional Modules  → Additional logic (e.g., Credit_Facility_v1)
```

### Authorization Models

**Role-Based (AUT_Roles_v2)**:

- Create named roles (e.g., "TRADER", "MANAGER")
- Assign roles to specific addresses
- Grant function permissions to roles
- Static membership until explicitly changed

**Token-Gated (AUT_TokenGated_Roles_v2)**:

- Roles based on token holdings
- Set thresholds (e.g., need 100 tokens for "HIGH_TIER" role)
- Dynamic membership as balances change
- Users automatically gain/lose roles

## E2E Test Structure

### Standard Test Template

```solidity
contract YourE2ETest is E2ETest {
    // Module configurations array - filled in setUp()
    IFloorFactory_v1.ModuleConfig[] moduleConfigurations;

    // Test actors - define roles
    address admin = makeAddr("admin");
    address user1 = makeAddr("user1");

    // Implement required abstract function
    function _configureSegments() internal override returns (PackedSegment[] memory) {
        // Define bonding curve segments
    }

    function setUp() public override {
        super.setUp();

        // Configure modules in order: Floor → Authorizer → Treasury
    }

    function test_e2e_YourScenario() public {
        // 1. Initialize Floor
        // 2. Setup Roles & Permissions
        // 3. Execute Business Logic
        // 4. Verify State & Access Control
    }
}
```

### Essential Testing Patterns

**1. Foundry Test Helpers**

```solidity
// Execute single call as another address
vm.prank(manager);
bondingCurve.enableBuy();

// Execute multiple calls as another address
vm.startPrank(trader);
{
    token.approve(address(bondingCurve), amount);
    bondingCurve.buy(amount, 1);
}
vm.stopPrank();

// Assert that next call reverts with specific error
vm.expectRevert(
    abi.encodeWithSelector(IModule_v2.Module__CallerNotPermissioned.selector)
);
bondingCurve.setBuyFee(200);

// Use scoped blocks { } to prevent "stack too deep" errors
```

**2. Token Setup**

```solidity
// Mint tokens to test actors
token.mint(trader, 100e18);

// Users must approve spending before trading
vm.prank(trader);
token.approve(address(bondingCurve), 100e18);
```

**3. Module References**

```solidity
// Floor address is the bonding curve
address floor = _create_E2E_Floor(...);
BC_Discrete_Redeeming_VirtualSupply_v1 bondingCurve =
    BC_Discrete_Redeeming_VirtualSupply_v1(floor);

// Get other modules from bonding curve
AUT_Roles_v2 authorizer = AUT_Roles_v2(bondingCurve.authorizer());
```

### Test Function Flow

Every E2E test follows this pattern:

```
Floor Initialization
    └── Create floor with all modules
    └── Get module references
    └── Setup minting permissions

Setup Roles & Permissions
    └── Create roles with descriptive names
    └── Assign roles to users
    └── Grant function permissions to roles

Execute Business Logic
    └── Enable trading/functionality
    └── Users execute operations (buy, sell, manage fees, etc.)
    └── Test role-based access

Verify & Test Access Control
    └── Assert expected state changes
    └── Test authorized users can act
    └── Test unauthorized users are blocked
```

## Best Practices

### 1. Use Section Dividers

```solidity
//--------------------------------------------------------------------------
// Floor Initialization
//--------------------------------------------------------------------------

//--------------------------------------------------------------------------
// Setup Roles & Permissions
//--------------------------------------------------------------------------

//--------------------------------------------------------------------------
// Execute Business Logic
//--------------------------------------------------------------------------
```

Consistent 78-character dividers make tests scannable.

### 2. Test Complete Workflows

Don't just test that functions work - test realistic scenarios:

- Manager enables trading → Trader buys → Trader sells
- User upgrades role → Gains new permissions → Can execute new functions
- Admin assigns role → Revokes role → User loses access

### 3. Make Tests Readable

E2E tests serve as documentation. Other developers should be able to read your test and understand:

- How to set up a Floor
- How to create and assign roles
- How to grant permissions
- What users can and cannot do

## Setup Checklist

When writing a new E2E test:

**Test Structure:**

- [ ] Extends `E2ETest`
- [ ] Implements `_configureSegments()` abstract function
- [ ] Calls `super.setUp()` in `setUp()`
- [ ] Defines test actors with `makeAddr()`
- [ ] Declares `IFloorFactory_v1.ModuleConfig[] moduleConfigurations`

**Module Configuration:**

- [ ] Modules configured in correct order (Floor, Authorizer, Treasury, Optional)
- [ ] Uses setup helpers from `E2EModuleRegistry` (e.g., `setUpRoleAuthorizer()`)
- [ ] Pushes configs to `moduleConfigurations` array

**Test Function:**

- [ ] Has section dividers for organization
- [ ] Grants minter role: `issuanceToken.setMinter(address(bondingCurve), true)`
- [ ] Enables trading if needed: `enableBuy()`, `enableSell()`
- [ ] Creates roles with descriptive names
- [ ] Documents role purposes with comments
- [ ] Grants permissions before testing access
- [ ] Tests both authorized and unauthorized access
- [ ] Verifies state changes and events
- [ ] Uses `console.log()` for important milestones

## Testing Strategy

### What E2E Tests Should Cover

**✅ Test:**

- Complete workflows (create floor → setup roles → execute operations)
- Integration between modules (authorizer checks, treasury fees, etc.)
- Role-based access control (who can call what)
- State changes from user perspective
- Realistic scenarios and user journeys

**❌ Don't Test:**

- Internal function logic (that's unit testing)
- Individual module functionality in isolation (that's integration testing)
- Edge cases of calculations (that's unit testing)

## Examples

### Complete Examples

**Role-Based Authorization:**
See `test/e2e/core/authorizer/RoleAuthorizerE2E.t.sol`

- Demonstrates creating roles
- Shows bulk permission assignment
- Tests role revocation
- Verifies admin role transfer

**Token-Gated Authorization:**
See `test/e2e/core/authorizer/TokenGatedRoleAuthorizerE2E.t.sol`

- Shows token-gated role setup
- Demonstrates role upgrades based on token holdings
- Tests tiered access (low/mid/high tier)
- Verifies dynamic permission changes

### Running E2E Tests

```bash
# Run all E2E tests
forge test --match-path "test/e2e/**/*.sol" -vv

# Run specific E2E test
forge test --match-test test_e2e_RoleAuthorizer -vv

# Run with very verbose output for debugging
forge test --match-test test_e2e_YourTest -vvvv
```

## Key Principles

### 1. E2E Tests Are Documentation

Other developers should be able to read your E2E test and understand:

- How the system works
- How to use different features
- What roles exist and what they can do
- Expected behaviors and restrictions

### 2. Simulate Real Usage

Think like a user:

- What would they do first?
- What permissions do they need?
- What can go wrong?
- What should they not be able to do?

### 3. Focus on Integration

E2E tests verify that:

- Modules work together correctly
- Authorization is enforced
- State changes propagate properly
- Events are emitted
- Workflows complete successfully

### 4. Keep It Maintainable

- Use descriptive names
- Add explanatory comments
- Organize with section dividers
- Follow consistent patterns
- Reference other tests for examples

## Summary

E2E tests for the Floor architecture verify complete workflows by testing multiple modules working together. They focus on:

- **Integration** over isolation
- **Workflows** over individual functions
- **Authorization** patterns and access control
- **Real scenarios** that users will encounter

Follow the patterns established in existing E2E tests, maintain clear documentation within tests, and always verify both authorized and unauthorized access paths.
