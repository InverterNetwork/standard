# Test Skeleton Structure

## Overview
Unit tests follow a consistent skeleton structure with organized sections, structured comments, and proper setup.

## File Structure

### 1. Header
```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
```

### 2. Imports (Organized by Category)
```solidity
// External Libraries
import {Clones} from "@oz/proxy/Clones.sol";
import {IERC20} from "@oz/token/ERC20/IERC20.sol";

// Internal Dependencies
import {ModuleTest, IModule_v2} from "@unitTest/core/modules/ModuleTest.sol";

// SuT (System under Test)
import {ContractName_Exposed} from "@mocks/path/ContractName_Exposed.sol";

// Errors
import {OZErrors} from "@testUtilities/OZErrors.sol";

// Mocks
import {MockContract} from "@mocks/path/MockContract.sol";
```

### 3. Contract Declaration
```solidity
contract ContractName_Test is ModuleTest {
```

### 4. State Variables
```solidity
    ///////////////////////////////////////////////////////////////////////////
    // State

    // SuT
    ContractName_Exposed _contractName;

    // Contracts
    DependencyContract _dependency;

    // Constants
    string private constant NAME = "Value";
    uint private constant AMOUNT = 100;
```

### 5. Setup Function
```solidity
    ///////////////////////////////////////////////////////////////////////////
    // Setup

    function setUp() public {
        address impl = address(new ContractName_Exposed(address(_forwarder)));
        _contractName = ContractName_Exposed(Clones.clone(impl));

        _setUpFloor();

        // Initialize dependencies
        _contractName.init(
            address(_floor),
            address(_authorizer),
            address(_feeTreasury),
            bytes("")
        );

        // Set permissions
        _authorizer.setAllAuthorized(true);
    }
```

### 6. Test Sections

#### a. Test Initialization
```solidity
    ///////////////////////////////////////////////////////////////////////////
    // Test Initialization

    /*
    Test: SupportsInterface
    └── Given: The interfaceId is IContractName
        └── When: the function supportsInterface is called
            └── Then: the function should return true
    */
    function testSupportsInterface() public override(ModuleTest) {
        assertTrue(
            _contractName.supportsInterface(type(IContractName).interfaceId)
        );
    }

    /*
    Test: Init
    └── When: the function init is called
        └── Then: [Expected behavior]
    */
    function testInit() public override {
        // Test implementation
    }

    /*
    Test: ReinitFails
    └── When: the function init is called after initialization
        └── Then: the function should revert
    */
    function testReinitFails() public override {
        vm.expectRevert(OZErrors.Initializable__InvalidInitialization);
        _contractName.init(/* params */);
    }
```

#### b. Test Modifiers

**Note:** Create a placeholder (`/* Test: modifierName */`) for **each modifier** in the contract to ensure tests are positioned correctly. Gherkin scenarios are added during implementation following guidance from `@test-modifiers.mdc`.

```solidity
    /////////////////////////////////////////////////////////////////////////////
    // Test Modifier

    /* Test: modifierName */
```

#### c. Test External Functions

**Note:** Create a placeholder (`/* Test: functionName */`) for **each external/public function** in the contract, organized by category, to ensure tests are positioned correctly. Gherkin scenarios are added during implementation following guidance from relevant MDC files (e.g., `@test-external-functions.mdc`).

```solidity
    ///////////////////////////////////////////////////////////////////////////
    // Test External Functions

    //===========================================================================
    // Public Functions - Getter Functions

    /* Test: getterFunction */
    // Trivial (if only returns state)

    //==========================================================================
    // Public Functions - Mutating Functions

    //--------------------------------------------------------------------------
    // Function Category

    /* Test: functionName */
```

#### d. Test Internal Functions

**Note:** Create a placeholder (`/* Test: _internalFunction */`) for **each internal function** in the contract, organized by category, to ensure tests are positioned correctly. Gherkin scenarios are added during implementation following guidance from `@test-internal-functions.mdc`.

```solidity
    ///////////////////////////////////////////////////////////////////////////
    // Test Internal Functions

    // ========================================================================
    // Internal Functions

    // -------------------------------------------------------------------------
    // Internal - Function Category

    /* Test: _internalFunction */
```

### 7. Helper Functions
```solidity
    ///////////////////////////////////////////////////////////////////////////
    // Helper Functions
}
```

## Exposed Mock Contract

Create an exposed version in `test/mocks/` to test internal functions:

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.0;

import {ContractName} from "src/path/ContractName.sol";

contract ContractName_Exposed is ContractName {
    constructor(address forwarder) ContractName(forwarder) {}

    function exposed_internalFunction(/* params */) external {
        _internalFunction(/* params */);
    }
}
```

## Naming Conventions

- Test contract: `ContractName_Test`
- Test functions: `testFunctionName_Condition` or `test_InternalFunction_Condition`
- Exposed contract: `ContractName_Exposed`
- Exposed functions: `exposed_internalFunction`
- State variables: `_variableName`
- Function parameters: `paramName_` (trailing underscore)

## Section Order

1. Test Initialization (supportsInterface, init, reinit)
2. Test Modifiers
3. Test External Functions
   - Public Getters
   - Public Mutating Functions (by category)
4. Test Internal Functions (by category)
5. Helper Functions

## Helper Functions in ModuleTest

The base `ModuleTest` class (`test/unit/core/modules/ModuleTest.sol`) provides common helper functions:

### Reentrancy Guard Initialization Check

For contracts using OpenZeppelin's `ReentrancyGuardUpgradeable`:

```solidity
/// @notice Asserts that the reentrancy guard was properly initialized.
/// @dev    Verifies the storage slot contains NOT_ENTERED (1), not
///         uninitialized (0). Use this in testInit() to verify
///         __ReentrancyGuard_init() was called.
/// @param  target_ The contract address to check.
function _assertReentrancyGuardInitialized(address target_) internal view {
    // OpenZeppelin ReentrancyGuardUpgradeable storage slot
    bytes32 slot =
        0x9b779b17422d0df92223018b32b4d1fa46e071723d6817e2486d003becc55f00;

    // Use vm.load to read target contract's storage
    uint status = uint(vm.load(target_, slot));

    // NOT_ENTERED = 1, uninitialized = 0
    require(status == 1, "Reentrancy guard not initialized");
}
```

### Usage Example

**In `testInit()` - verify reentrancy guard initialization:**

```solidity
function testInit() public override {
    // ... other init checks ...

    // Verify reentrancy guard is initialized (storage slot = NOT_ENTERED = 1)
    _assertReentrancyGuardInitialized(address(_contract));
}
```
