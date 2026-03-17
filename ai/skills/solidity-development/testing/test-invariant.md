# Invariant Testing Workflow

## Overview

Invariant tests verify that critical protocol properties hold under randomized sequences of operations. Unlike unit tests (isolated components) or E2E tests (scripted workflows), invariant tests use fuzzing to explore state space and discover edge cases.

**Key Concepts:**
- **Invariants**: Properties that must always be true (e.g., "collateral must back supply")
- **Handlers**: Fuzzer-friendly wrappers that prevent reverts and track state
- **Ghost Variables**: Cumulative state tracked in handlers for verification
- **fail_on_revert**: Enforces that handlers prevent all reverts (no wasted fuzzing cycles)

**Testing Strategy:**
1. **Identify invariants** from stakeholder perspectives
2. **Build handlers** that wrap operations safely
3. **Write invariant checks** that verify properties
4. **Run with depth** to find multi-step bugs

## Prerequisites

- `InvariantTest.sol` base contract exists (inherits `ModuleRegistry`)
- `InvariantHandlerBase.sol` provides common utilities
- Handler structure mirrors `src/` directory layout
- Foundry configuration set with `[invariant]` section

## File Structure

### Handler Location

Handlers MUST mirror the `src/` structure:

```
src/core/authorizer/AUT_Roles_v2.sol
→ test/invariant/handlers/core/authorizer/AUT_Roles_v2_InvariantHandler.sol

src/core/issuance/BC_Discrete_Redeeming_VirtualSupply_v1.sol
→ test/invariant/handlers/core/issuance/BC_Discrete_v1_InvariantHandler.sol
```

### Test Location

Tests mirror `src/` structure in `test/invariant/`:

```
test/invariant/core/authorizer/AUT_Roles_v2_InvariantTest.t.sol
test/invariant/core/issuance/BC_Discrete_v1_InvariantTest.t.sol
```

## High-Level Workflow

### Phase 1: Identify Invariants (Stakeholder Perspective)

#### Step 1: List Stakeholders

Identify who interacts with the contract:
- **End users**: What do they expect?
- **Admins**: What security properties must hold?
- **Protocol**: What mathematical relationships are critical?
- **Other contracts**: What guarantees do integrators need?

#### Step 2: Ask Expectation Questions

For each stakeholder, ask:
- **Functional**: What should always work?
- **Security**: What should never happen?
- **Mathematical**: What relationships must hold?
- **State**: What must always be true about state?

#### Step 3: Document Invariants

Write down each invariant with clear statement:

```solidity
/*
Invariant: default_admin_always_has_permission
Stakeholder: Protocol security
Property: DEFAULT_ADMIN_ROLE must have permission to call any function
Why: Admin role must never lose authority
*/
```

### Phase 2: Build Handler

#### Step 4: Create Handler Contract

**File**: `test/invariant/handlers/core/category/ContractName_InvariantHandler.sol`

**Template:**
```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.0;

import "forge-std/console.sol";

// Internal Dependencies
import {InvariantHandlerBase} from
    "@invariant/handlers/InvariantHandlerBase.sol";
import {ContractName} from "@category/ContractName.sol";

contract ContractName_InvariantHandler is InvariantHandlerBase {
    //--------------------------------------------------------------------------
    // State Variables
    //--------------------------------------------------------------------------

    /// @notice The contract being tested
    ContractName public contractName;

    /// @notice Additional contracts or constants if needed
    address public immutable targetContract;

    //--------------------------------------------------------------------------
    // Ghost Variables
    //--------------------------------------------------------------------------

    /// @notice Track created entities
    uint256[] internal _createdIds;

    /// @notice Track actors
    address[] internal _actors;
    mapping(address => bool) internal _isKnownActor;

    /// @notice Track operation counts
    uint256 public totalOperations;

    //--------------------------------------------------------------------------
    // Constructor
    //--------------------------------------------------------------------------

    constructor(ContractName contractName_, address targetContract_) {
        contractName = contractName_;
        targetContract = targetContract_;

        // Initialize with default actors
        _addActor(address(this));
        _addActor(makeAddr("alice"));
    }

    //--------------------------------------------------------------------------
    // Handler Functions
    //--------------------------------------------------------------------------

    /// @notice Handler functions go here
    /// Each wraps a contract operation in fuzzer-friendly way
}
```

#### Step 5: Implement Handler Functions

**Critical Rules:**
- ✅ Return early if preconditions fail (avoid reverts)
- ✅ Bound all inputs to realistic ranges
- ✅ Track state in ghost variables
- ✅ Use `vm.prank()` for different callers
- ❌ Never let operations revert

**Pattern A: Create/Add Operations**
```solidity
function createEntity(uint256 seed) external {
    // Limit to reasonable count
    if (_createdIds.length >= 100) return;

    totalOperations++;

    // Bound seed to avoid overflow
    seed = bound(seed, 0, type(uint128).max);

    // Create parameters
    address actor = _createActor(seed);
    uint256 value = _boundNonZeroAmount(seed);

    // Get appropriate caller
    address caller = _getValidCaller(seed);

    // Execute with try/catch
    vm.prank(caller);
    try contractName.create(value) returns (uint256 id) {
        _createdIds.push(id);
        _addActor(actor);
    } catch {
        return; // Early exit on failure
    }
}
```

**Pattern B: Modify Existing State**
```solidity
function modifyEntity(uint256 idSeed, uint256 valueSeed) external {
    // Need existing entities
    if (_createdIds.length == 0) return;

    totalOperations++;

    // Select existing entity
    uint256 index = idSeed % _createdIds.length;
    uint256 id = _createdIds[index];

    // Bound new value
    uint256 newValue = _boundNonZeroAmount(valueSeed);

    // Get appropriate caller
    address caller = _getValidCaller(idSeed);

    // Execute
    vm.prank(caller);
    try contractName.modify(id, newValue) {
        // Update ghost variables if needed
    } catch {
        return;
    }
}
```

**Pattern C: Remove/Delete Operations**
```solidity
function removeEntity(uint256 seed) external {
    if (_createdIds.length == 0) return;

    totalOperations++;

    // Select entity to remove
    uint256 index = seed % _createdIds.length;
    uint256 id = _createdIds[index];

    address caller = _getValidCaller(seed);

    vm.prank(caller);
    try contractName.remove(id) {
        // Remove from ghost tracking
        _createdIds[index] = _createdIds[_createdIds.length - 1];
        _createdIds.pop();
    } catch {
        return;
    }
}
```

#### Step 6: Add View Functions for Invariants

Expose ghost variables for invariant checking:

```solidity
//--------------------------------------------------------------------------
// View Functions for Invariant Checking
//--------------------------------------------------------------------------

/// @notice Returns all created entity IDs
function getCreatedIds() external view returns (uint256[] memory) {
    return _createdIds;
}

/// @notice Returns all known actors
function getActors() external view returns (address[] memory) {
    return _actors;
}

/// @notice Returns operation count
function getTotalOperations() external view returns (uint256) {
    return totalOperations;
}
```

#### Step 7: Add Helper Functions

Internal helpers for common operations:

```solidity
//--------------------------------------------------------------------------
// Internal Helper Functions
//--------------------------------------------------------------------------

function _addActor(address actor_) internal {
    if (!_isKnownActor[actor_]) {
        _actors.push(actor_);
        _isKnownActor[actor_] = true;
    }
}

function _getValidCaller(uint256 seed_) internal view returns (address) {
    if (_actors.length == 0) return address(this);
    return _actors[seed_ % _actors.length];
}
```

### Phase 3: Write Invariant Test

#### Step 8: Create Test Contract

**File**: `test/invariant/core/category/ContractName_InvariantTest.t.sol`

**Template:**
```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.0;

import "forge-std/console.sol";

// Internal Dependencies
import {InvariantTest, IFloorFactory_v1} from "@invariant/InvariantTest.sol";
import {ContractName_InvariantHandler} from
    "@invariant/handlers/core/category/ContractName_InvariantHandler.sol";

// SuT
import {ContractName} from "@category/ContractName.sol";

// Other required imports...

contract ContractName_InvariantTest is InvariantTest {
    //--------------------------------------------------------------------------
    // State Variables
    //--------------------------------------------------------------------------

    /// @notice The handler for operations
    ContractName_InvariantHandler public handler;

    /// @notice The contract being tested
    ContractName public contractName;

    /// @notice Module configurations
    IFloorFactory_v1.ModuleConfig[] moduleConfigurations;

    //--------------------------------------------------------------------------
    // Setup
    //--------------------------------------------------------------------------

    function _configureSegments()
        internal
        override
        returns (PackedSegment[] memory segments)
    {
        // Implement if testing bonding curves
        // Otherwise leave simple default
        segments = new PackedSegment[](1);
        segments[0] = PackedSegmentLib._create(1e18, 0, 1000e18, 1);
    }

    function setUp() public override {
        super.setUp();

        // Setup modules (if needed)
        setUpRoleAuthorizer();
        setUpSplitterTreasury();
        // ... other modules

        // Configure module array
        // moduleConfigurations[0] => Floor
        // moduleConfigurations[1] => Authorizer
        // moduleConfigurations[2] => Treasury

        // Create floor (if testing a module)
        address floor = _createFloorWithModules(
            moduleConfigurations[0],
            moduleConfigurations[1],
            moduleConfigurations[2],
            new IFloorFactory_v1.ModuleConfig[](0)
        );

        contractName = ContractName(floor); // or however you get it

        // Deploy handler
        handler = new ContractName_InvariantHandler(
            contractName,
            targetContractAddress
        );

        // Grant permissions if needed
        // e.g., authorizer.grantRole(roleId, address(handler));

        // Target only the handler
        targetContract(address(handler));

        // Define which handler functions can be called
        bytes4[] memory selectors = new bytes4[](3);
        selectors[0] = ContractName_InvariantHandler.createEntity.selector;
        selectors[1] = ContractName_InvariantHandler.modifyEntity.selector;
        selectors[2] = ContractName_InvariantHandler.removeEntity.selector;

        targetSelector(
            FuzzSelector({addr: address(handler), selectors: selectors})
        );
    }

    //--------------------------------------------------------------------------
    // Invariants
    //--------------------------------------------------------------------------

    /// @notice [Invariant description]
    /// @dev [Technical details and why this matters]
    function invariant_propertyDescription() public view {
        // Get data from contract and handler
        uint256[] memory ids = handler.getCreatedIds();

        // Check the invariant
        for (uint256 i; i < ids.length; i++) {
            // Verify property
            assertTrue(/* condition */, "Description of what must be true");
        }
    }

    //--------------------------------------------------------------------------
    // Helper Function for Debugging
    //--------------------------------------------------------------------------

    function logHandlerState() external view {
        console.log("=== Handler State ===");
        console.log("Total operations:", handler.getTotalOperations());
        console.log("Created entities:", handler.getCreatedIds().length);
        console.log("Known actors:", handler.getActors().length);
    }
}
```

#### Step 9: Implement Invariants

For each invariant identified in Phase 1:

```solidity
/// @notice [Property that must always hold]
/// @dev [Why this is critical, what it prevents]
function invariant_descriptiveName() public view {
    // 1. Get necessary data
    uint256[] memory ids = handler.getCreatedIds();
    address[] memory actors = handler.getActors();

    // 2. Check the property
    for (uint256 i; i < ids.length; i++) {
        // Get state from contract
        (uint256 value, address owner, bool active) = 
            contractName.getEntity(ids[i]);

        // Assert property
        assertTrue(active, "All tracked entities must be active");
        assertTrue(owner != address(0), "All entities must have valid owner");

        // Cross-check with ghost variables if applicable
    }

    // 3. Check global properties
    assertGe(
        contractName.totalValue(),
        handler.getTotalOperations(),
        "Total value must be at least operation count"
    );
}
```

### Phase 4: Configuration & Execution

#### Step 10: Verify foundry.toml Configuration

Ensure `foundry.toml` has:

```toml
[invariant]
runs = 16           # Test sequences (local dev)
depth = 512         # Function calls per sequence
fail_on_revert = true  # Enforce no reverts in handlers

[profile.ci]
invariant = { runs = 32, depth = 1024 }  # Deeper testing in CI
```

**Settings explained:**
- **runs**: How many independent test sequences (16 for local speed)
- **depth**: Function calls per sequence (512 = 8,192 total calls)
- **fail_on_revert**: Forces handlers to prevent all reverts (better handlers)

#### Step 11: Run Tests

**Local development:**
```bash
forge test --match-path "test/invariant/**/*.sol"
```

**With specific test:**
```bash
forge test --match-test "invariant_propertyName"
```

**CI profile (deeper):**
```bash
FOUNDRY_PROFILE=ci forge test --match-path "test/invariant/**/*.sol"
```

#### Step 12: Verify Results

Check test output:

```
[PASS] invariant_propertyName() (runs: 16, calls: 8192, reverts: 0)

╭─────────────────────+──────────────+───────+─────────+──────────╮
| Contract            | Selector     | Calls | Reverts | Discards |
+=====================================================================+
| YourHandler         | createEntity | 1200  | 0       | 0        |
| YourHandler         | modifyEntity | 1150  | 0       | 0        |
╰─────────────────────+──────────────+───────+─────────+──────────╯
```

**Good signs:**
- ✅ Reverts: 0 (handlers prevent all reverts)
- ✅ Discards: 0 (no wasted fuzzing cycles)
- ✅ Balanced calls (fuzzer exploring evenly)

**Bad signs:**
- ❌ High reverts (handler not checking preconditions)
- ❌ Uneven calls (some functions not being called)

## Best Practices

### Handler Design

**DO:**
- ✅ Check preconditions and return early
- ✅ Bound all inputs to realistic ranges (use `bound()`)
- ✅ Track important state in ghost variables
- ✅ Use `try/catch` for operations that might fail
- ✅ Limit collection sizes (e.g., max 100 entities)
- ✅ Use `vm.prank()` to test from different actors
- ✅ Restrict actor address space (uint16.max = 65k addresses)

**DON'T:**
- ❌ Let operations revert (use `fail_on_revert = true`)
- ❌ Create unbounded loops
- ❌ Use entire uint256 range for addresses
- ❌ Forget to track state in ghost variables
- ❌ Include view functions in selectors (they don't change state)

### Invariant Design

**Good invariants are:**
- **Specific**: Clear property that can be checked
- **Universal**: Must hold after ANY sequence of operations
- **Checkable**: Can be verified in reasonable time
- **Meaningful**: Catches real bugs if violated

**Examples:**

✅ **Good**: "Sum of all user balances equals total supply"
- Specific, checkable, catches accounting bugs

✅ **Good**: "Virtual collateral always backs issued tokens at curve price"
- Specific, universal, prevents insolvency

❌ **Bad**: "System works correctly"
- Too vague, not checkable

❌ **Bad**: "No unexpected reverts"
- Already enforced by `fail_on_revert`

### Ghost Variables

**When to use:**
- Track entities created (IDs, addresses)
- Count operations performed
- Store derived values not in contract
- Track relationships between entities
- Accumulate values over time

**Example patterns:**

```solidity
// Track created entities
uint256[] internal _entityIds;

// Track actors by role
mapping(bytes32 => address[]) internal _roleMembers;

// Count operations by type
mapping(bytes4 => uint256) internal _operationCounts;

// Accumulate values
uint256 internal _cumulativeValue;
```

### Search Space Optimization

**Problem**: Unrestricted fuzzing is inefficient (won't find bugs)

**Solutions:**

1. **Limit actors** - Use uint16.max instead of uint256.max
   ```solidity
   function _createActor(uint256 seed_) internal pure returns (address) {
       seed_ = bound(seed_, 1, type(uint16).max);
       return address(uint160(seed_));
   }
   ```

2. **Reuse actors** - Track created actors and reuse them
   ```solidity
   address[] internal _actors;
   function _getActor(uint256 seed_) internal view returns (address) {
       return _actors[seed_ % _actors.length];
   }
   ```

3. **Bound amounts** - Use realistic ranges
   ```solidity
   uint256 internal constant MAX_REALISTIC_AMOUNT = type(uint128).max;
   amount = bound(amount, 1, MAX_REALISTIC_AMOUNT);
   ```

4. **Limit collections** - Cap array sizes
   ```solidity
   if (_entities.length >= 100) return;
   ```

5. **Skip trivial operations** - Don't expose view functions as handler functions

## Complete Example: Authorization System

See `test/invariant/core/authorizer/AUT_Roles_v2_InvariantTest.t.sol` for complete example.

**Invariants tested:**
1. DEFAULT_ADMIN_ROLE always has permission
2. Role member counts match tracked state
3. Burned admin roles cannot change
4. Permissions imply role existence
5. PUBLIC_ROLE accessible by anyone
6. Role admins have authority
7. Role IDs increase monotonically
8. No orphaned permissions

**Handler includes:**
- Ghost variables for tracking roles, members, permissions
- Bounded actor creation (uint16.max)
- Precondition checks before every operation
- Try/catch for all state changes
- Helper functions for common patterns

## Debugging Failed Invariants

When an invariant fails, Foundry shows the call sequence:

```
[FAIL] invariant_propertyName()

Call sequence:
1. handler.createEntity(12345)
2. handler.modifyEntity(67890, 11111)
3. handler.removeEntity(22222)
```

**Debug steps:**

1. **Copy sequence to concrete test**
   ```solidity
   function test_ReproduceFailure() public {
       handler.createEntity(12345);
       handler.modifyEntity(67890, 11111);
       handler.removeEntity(22222);
       // Check what broke
   }
   ```

2. **Add logging in handler**
   ```solidity
   console.log("Creating entity:", id);
   console.log("State before:", someValue);
   ```

3. **Check ghost variables**
   ```solidity
   function logHandlerState() external view {
       console.log("Ghost variable 1:", _ghostVar1);
       console.log("Ghost variable 2:", _ghostVar2);
   }
   ```

4. **Compare contract state vs ghost state**
   - If mismatch, ghost tracking has bug
   - If match, invariant check has bug or real violation

## Integration with Other Test Types

### Relationship to Unit Tests

- **Unit tests**: Verify individual functions work correctly in isolation
- **Invariant tests**: Verify properties hold across random operation sequences

**Example:**
- Unit test: `testCreateRole_Works()` - Create role works with valid inputs
- Invariant: `invariant_role_ids_monotonic()` - Role IDs never decrease (tested across thousands of random creates)

### Relationship to E2E Tests

- **E2E tests**: Verify specific workflows (scripted user journeys)
- **Invariant tests**: Verify properties under any workflow (randomized)

**Example:**
- E2E: User creates floor → assigns roles → trades (specific sequence)
- Invariant: Collateral backs supply (checked after random buys/sells/reconfigs)

### Coverage

Invariant tests complement but don't replace unit/E2E tests:

- **Unit**: 100% branch coverage of function logic
- **E2E**: Real user workflows and integration
- **Invariant**: Edge cases from unexpected operation sequences

## Checklist

**Phase 1 - Identify Invariants:**
- [ ] List all stakeholders
- [ ] Ask expectation questions for each
- [ ] Document each invariant with stakeholder and why
- [ ] Prioritize critical invariants (start with 5-8)

**Phase 2 - Build Handler:**
- [ ] Create handler in `handlers/core/category/` (mirror src)
- [ ] Inherit from `InvariantHandlerBase`
- [ ] Define ghost variables for tracking
- [ ] Implement handler functions with try/catch
- [ ] Bound all inputs appropriately
- [ ] Add view functions to expose ghost variables
- [ ] Verify `fail_on_revert = true` in config

**Phase 3 - Write Tests:**
- [ ] Create test in `core/category/` (mirror src)
- [ ] Inherit from `InvariantTest`
- [ ] Setup floor and modules if needed
- [ ] Deploy handler with appropriate permissions
- [ ] Target handler contract
- [ ] Define selectors (exclude view functions)
- [ ] Implement invariant checks
- [ ] Add debug helper (logHandlerState)

**Phase 4 - Run & Verify:**
- [ ] Run `forge test --match-path "test/invariant/**/*.sol"`
- [ ] Check: All invariants pass
- [ ] Check: Reverts = 0
- [ ] Check: Calls distributed evenly
- [ ] Test with CI profile (deeper)
- [ ] Clean cached failures if bytecode changed

## Common Patterns

### Pattern: Multi-Contract Coordination

Test invariants across multiple contracts:

```solidity
contract MultiContract_InvariantHandler is InvariantHandlerBase {
    ContractA public contractA;
    ContractB public contractB;

    function operationSpanningBoth(uint256 seed) external {
        // Operations that affect both
        // Verify coordination in invariant checks
    }
}
```

### Pattern: Time-Based Invariants

Test properties that involve time:

```solidity
function advanceTime(uint256 seed) external {
    uint256 jump = bound(seed, 0, MAX_TIME_JUMP_MEDIUM);
    vm.warp(block.timestamp + jump);
}

function invariant_timeLocks_enforced() public view {
    // Check time-locked operations respected delays
}
```

### Pattern: Accumulated Value Tracking

Track cumulative values:

```solidity
uint256 internal _cumulativeFees;

function trade(uint256 amount) external {
    // ... operation
    _cumulativeFees += calculatedFee;
}

function invariant_fees_accumulate_correctly() public view {
    assertEq(treasury.totalFees(), _cumulativeFees);
}
```

## Resources

- [Invariant Testing Guide](../test/invariant/Everything%20is%20alright%20(hopefully)%20-%20Invariant%20Test%20703d05122a5a41dc97678d8483142387.md)
- [Foundry Invariant Testing](https://book.getfoundry.sh/forge/invariant-testing)
- Example: `test/invariant/core/authorizer/AUT_Roles_v2_InvariantTest.t.sol`
- Example Handler: `test/invariant/handlers/core/authorizer/AUT_Roles_v2_InvariantHandler.sol`

## Key Principles Summary

1. **Invariants are expectations** - From user, admin, protocol, integrator perspectives
2. **Handlers prevent reverts** - Use preconditions, try/catch, bounded inputs
3. **Ghost variables track state** - For checking properties not directly queryable
4. **Structure mirrors src/** - Handlers and tests follow source layout
5. **fail_on_revert enforces quality** - No wasted fuzzing on reverting operations
6. **Depth > runs** - For complex contracts, deeper sequences find more bugs
7. **Start simple, expand** - Begin with obvious invariants, add complexity
8. **Complement other tests** - Invariants catch what unit/E2E tests miss
