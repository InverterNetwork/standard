# Inverter Code Standard

Version 1.0.0-beta - Last Changed: 2024-12-06

The Inverter Code Standard is a comprehensive set of guidelines for developing software within the Inverter Network. This standard aims to ensure consistent code quality, security, and maintainability across all projects. For now, it is focused on Solidity smart contracts, but will be expanded to cover all aspects of development in the future.



## Table of Contents
1. [Introduction](#1-introduction)
   - [Purpose](#purpose)
   - [Scope](#scope)
   - [How to Use This Document](#how-to-use-this-document)

2. [General Principles](#2-general-principles)
   - [Code Quality](#code-quality)
   - [Security First](#security-first)
   - [Documentation Requirements](#documentation-requirements)

3. [Tools and Environment](#3-tools-and-environment)
   - [Required Tools](#required-tools)

4. [Naming Conventions](#4-naming-conventions)
   - [Contracts](#contracts)
   - [Functions](#functions)
   - [Variables](#variables)
   - [Files and Folders](#files-and-folders)

5. [Code Layout](#5-code-layout)
   - [Contract Structure](#contract-structure)
   - [Interface Structure](#interface-structure)

6. [Contract Inheritance](#6-contract-inheritance)
   - [Interface Inheritance](#interface-inheritance)
   - [Virtual Function Standards](#virtual-function-standards)
   - [Storage Layout Considerations](#storage-layout-considerations)
   - [Initialization Patterns](#initialization-patterns)

7. [Documentation Standards](#7-documentation-standards)
   - [NatSpec Requirements](#natspec-requirements)
   - [Folder-based READMEs](#folder-based-readmes)
   - [Code Comments](#code-comments)

8. [Testing](#8-testing)
   - [Test Types and Coverage](#test-types-and-coverage)
   - [Gherkin Test Documentation](#gherkin-test-documentation)
   - [Mock Usage](#mock-usage)
   - [Test Naming Conventions](#test-naming-conventions)

9. [Git Standards](#9-git-standards)
   - [Branch Strategy](#branch-strategy)
   - [Commit Messages](#commit-messages)
   - [Pull Request Process](#pull-request-process)

10. [Development Workflow](#10-development-workflow)
    - [Development Notes](#development-notes)
    - [Versioning](#versioning)
    - [Deployment Guidelines](#deployment-guidelines)

## 1. Introduction

### Purpose
The Inverter Code Standard serves as the definitive guide for code quality, development practices, and technical processes within the Inverter Network. This standard aims to:
- Establish consistent coding practices across all projects
- Ensure our security-first development approach
- Streamline code review processes
- Facilitate maintainable and scalable code
- Enable efficient onboarding of new developers

### Scope
This standard applies to:
- All smart contract development
- Associated testing suites
- Supporting infrastructure code
- Documentation and technical specifications
- Development and deployment processes

While initially focused on Solidity smart contracts, these standards provide a foundation for all future development work across different technologies, and will be expanded to cover all aspects of development in the future.

### How to Use This Document
1. **For New Developers**
   - Read through the entire document before starting development
   - Use as a reference during the development process

2. **For Reviewers**
   - Use as a baseline for code review criteria
   - Reference specific sections when providing feedback
   - Ensure all requirements are met before approval

3. **For Project Customization**
   - Adapt universal principles to your specific project needs
   - Maintain consistency with these universal standards

4. **Document Maintenance**
   - Updates require team consensus
   - Suggestions for improvements are welcome through PRs

5. **Enforcement**
   - All code must comply with these standards
   - Deviations require explicit approval and documentation
   - Automated tools will enforce applicable standards where possible
   - Regular code reviews will ensure continued compliance

## 2. General Principles

### Code Quality
- **Readability First**
  - Code should be self-documenting and easily understood
  - Complex logic must be clearly commented
  - Consistent formatting and style across all files
  - Meaningful variable and function names

- **Maintainability**
  - Follow DRY (Don't Repeat Yourself) principles
  - Design for future extensibility and modularity
  - Keep functions focused and concise
  - Limit function complexity and nesting

- **Modularity**
  - Use inheritance and interfaces appropriately
  - Create reusable components when possible
  - Minimize dependencies between contracts

### Security First
- **Core Principles**
  - Security takes precedence over convenience
  - All code must be written with security as the primary concern
  - Follow established security patterns and best practices
  - When in doubt, choose the more secure option

- **Security Requirements**
  - Implement robust access controls
  - Follow check-effects-interactions pattern

- **Known Vulnerabilities**
  - Regular security audits and reviews by external parties for any changes
  - Document all security considerations, also within the code if applicable

### Documentation Requirements
- **Code Documentation**
  - All public functions must have NatSpec comments
  - Complex algorithms need detailed explanations
  - Security considerations must be explicitly documented
  - Include usage examples for complex functionality

- **Project Documentation**
  - Comprehensive README files
  - Architecture diagrams and explanations
  - Deployment procedures and requirements
  - Environment setup instructions
  - Troubleshooting guides

- **Maintenance Documentation**
  - Known issues and workarounds
  - Deprecation notices when applicable

## 3. Tools and Environment

### Required Tools

**Development Stack**:
- **Foundry**: Primary development framework
- **VSCode**: Recommended IDE with Solidity extensions
- **Git**: Version control

**Recommended Extensions**:
- Inline Bookmarks (for @todo/@note tracking)
- Solidity language support
- GitLens for enhanced Git integration

**AI Tooling** (Optional but Recommended):
- Cursor


## 4. Naming Conventions

### Contracts

**Contract Files**
- End with their major version: `ContractName_v1`
- Use PascalCase for contract names
- Example: `BountyManager_v1`

**Test Files**
- Copy contract name and add `.t.sol`: `BountyManager_v1.t.sol`
- Test contracts add `_Test`: `BountyManager_v1_Test`

**Mock Files**
- Access mocks (for internal functions): `ContractName_v1_Exposed.sol`
- Regular mocks: `ContractName_v1_Mock.sol`

### Functions

**Visibility Patterns**
- Private & Internal functions start with `_`
- Example: `_calculateFee()`

**Getter Functions**
- Boolean getters: `is<StateInTrueWay>()` - Example: `isOpen()`
- General getters: `get<VariableName>()` - Example: `getFundingManager()`
- Mapping getters: `get<Target>By<Parameter>()` - Example: `getBountyById()`

**Modifiers**
- Named as: `contextIsState()` - Example: `userIsAuthorized()`
- Internal modifier functions: `_ensureContextIsState` - Example: `_ensureUserIsAuthorized()`

### Variables

**Parameters**
- Always add `_` at the end: `user_`, `amount_`
- Applies to function parameters, event parameters, and error parameters
- Exception: Mapping key/value names do NOT use trailing underscores

**Return Values**
- Always name return values: `returns (uint amount_)`
- Always explicitly return, no implicit assignment
- Example:
  ```solidity
  /// @param y_ this is the input
  /// @return x_ this is returned
  function example(uint y_) returns (uint x_) {
      if(y_ > 2) {
          x_ = 15;
      } else {
          x_ = 16;
      }
      return x_;
  }
  ```

**State Variables**
- Internal/Private start with `_`: `_emergencyState`
- Everything internal by default unless good reason
- Constants can be public (must be in interface)

**Local Variables**
- No underscores at start or end
- Use camelCase: `totalAmount`, `userBalance`, `isValid`

**Events and Errors**
- Event parameter names: Use trailing `_` like function parameters
- Error parameter names: Use trailing `_` like function parameters
- Examples:
  ```solidity
  event PaymentProcessed(address indexed user_, uint amount_, bytes32 txHash_);
  error InvalidAmount(uint provided_, uint minimum_);
  ```

**Mappings**
- Always name mapping parameters (no trailing underscores for key/value names)
- Pattern: `mapping(<keyType> <keyName> => <valueType> <valueName>) internal _<descriptiveName>`
- Examples:
  - `mapping(uint id => Bounty bounty) internal _idToBounty`
  - `mapping(address user => uint balance) internal _userBalances`
  - `mapping(bytes32 role => bool hasRole) internal _rolePermissions`

**Underscore Usage Summary**
- **Leading `_`**: Private/internal functions and state variables
- **Trailing `_`**: Function parameters, event parameters, error parameters, return values
- **No underscores**: Local variables, mapping key/value names, public functions
- **Examples**:
  ```solidity
  contract Example {
      uint internal _balance;                           // Leading: internal variable

      function process(uint amount_) returns (bool success_) {  // Trailing: params & returns
          uint localAmount = amount_ * 2;               // None: local variable
          return _processInternal(localAmount);         // Leading: internal function
      }

      function _processInternal(uint value) internal { ... }    // Leading: internal function

      mapping(address user => uint balance) _userBalances;      // Mixed: trailing for variable, none for mapping params
  }
  ```

### Files and Folders

**File Naming**
- Files use PascalCase: `PaymentProcessor_v1.sol`
- Folders use camelCase: `paymentProcessor/`
- File names should match primary contract within
- Interfaces prefixed with "I": `IPaymentProcessor_v1.sol`

## 5. Code Layout

### Contract Structure

All contracts follow this order:

1. **Licensing**: `// SPDX-License-Identifier: LGPL-3.0-only`
2. **Pragma**:
   - **Contracts**: `pragma solidity 0.8.23;` (fixed version for production deployments)
   - **Interfaces**: `pragma solidity ^0.8.0;` (flexible version for broader compatibility)
   - **Rationale**: Contracts use fixed versions to ensure deterministic deployments and prevent unexpected behavior from compiler updates. Interfaces use flexible versions to allow integration with projects using different Solidity versions within the 0.8.x range.
3. **Imports**:
   ```solidity
   // External
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

   // Internal
   import "./interfaces/IPaymentProcessor_v1.sol";
   ```
4. **Contract Overview Documentation** (see Documentation Standards)
5. **Contract Declaration with Inheritance**
6. **Using Libraries**: `using SafeERC20 for IERC20;`
7. **ERC165 Interface Support**
8. **Constants**
9. **Storage Variables**
10. **Modifiers**
11. **Constructor** (modules typically don't use constructors)
12. **Initialization Functions**
13. **Public Functions** (grouped by getter/mutating, sorted by functionality)
14. **Internal Functions** (plain internal, then overridden)

**Example Structure:**

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity 0.8.23;

// Internal Dependencies
import {Module_v2, IModule_v2} from "@mod/base/Module_v2.sol";

// External Libraries
import {SafeERC20} from "@oz/token/ERC20/utils/SafeERC20.sol";

/**
 * @title   Example Contract
 *
 * @notice  Brief description of contract functionality
 *
 * @custom:security-contact security@inverter.network
 * @custom:version 2.0.0
 * @author  Inverter Network
 */
abstract contract ExampleContract_v2 is Module_v2 {
    using SafeERC20 for IERC20;

    // ========================================================================
    // Storage

    // ------------------------------------------------------------------------
    // Constants

    uint public constant BPS = 10_000;

    // ------------------------------------------------------------------------
    // State Variables

    IERC20 internal _token;
    bool public _isOpen;

    // ------------------------------------------------------------------------
    // Storage Gap

    uint[50] private __gap;

    // ========================================================================
    // Modifiers

    modifier onlyWhenOpen() {
        if (!_isOpen) revert FunctionalityClosed();
        _;
    }

    // ========================================================================
    // Public Functions - Getter Functions

    // ------------------------------------------------------------------------
    // Token Related Getters

    function getToken() external view returns (address) {
        return address(_token);
    }

    // ------------------------------------------------------------------------
    // State Related Getters

    function isOpen() external view returns (bool) {
        return _isOpen;
    }

    // ========================================================================
    // Public Functions - Mutating Functions

    // ------------------------------------------------------------------------
    // Processing Functions

    function process(uint amount_) external permissioned onlyWhenOpen {
        _processInternal(amount_);
    }

    // ------------------------------------------------------------------------
    // Administrative Functions

    function setOpen(bool isOpen_) external permissioned {
        _isOpen = isOpen_;
    }

    // ========================================================================
    // Internal Functions

    // ------------------------------------------------------------------------
    // Processing Logic

    function _processInternal(uint amount_) internal virtual {
        // Implementation
    }

    // ------------------------------------------------------------------------
    // Validation Logic

    function _validateAmount(uint amount_) internal pure {
        if (amount_ == 0) revert InvalidAmount();
    }
}
```

### Interface Structure

1. **Licensing**: `// SPDX-License-Identifier: LGPL-3.0-only`
2. **Pragma**: `pragma solidity ^0.8.0;`
3. **Imports** (External, then Internal)
4. **Contract Overview Documentation**
5. **Interface Declaration with Inheritance**
6. **Structs**
7. **Errors**
8. **Events**
9. **Functions** (getter, then mutating, sorted by functionality)

**Example Structure:**

```solidity
// SPDX-License-Identifier: LGPL-3.0-only
pragma solidity ^0.8.0;

/**
 * @title   IExampleContract_v2
 *
 * @notice  Interface for the Example contract's external API
 *
 * @custom:security-contact security@inverter.network
 * @custom:version 2.0.0
 * @author  Inverter Network
 */
interface IExampleContract_v2 {
    // ========================================================================
    // Structs

    /// @notice The processing configuration properties.
    /// @param  token  The token contract address used for processing.
    /// @param  feeRate  The fee rate expressed in basis points.
    /// @param  minAmount  The minimum amount required for processing.
    /// @param  maxAmount  The maximum amount allowed for processing.
    /// @param  processingIsOpen  The indicator used for enabling/disabling
    ///         the processing functionalities.
    /// @param  autoCompound  The indicator for automatic compounding of
    ///         rewards.
    struct ProcessingProperties {
        address token;
        uint feeRate;
        uint minAmount;
        uint maxAmount;
        bool processingIsOpen;
        bool autoCompound;
    }

    // ========================================================================
    // Errors

    // ------------------------------------------------------------------------
    // Validation Errors

    /// @notice Amount provided was invalid
    error Module__ExampleContract__InvalidAmount();

    /// @notice Receiver address provided was invalid
    error Module__ExampleContract__InvalidReceiver();

    // ------------------------------------------------------------------------
    // State Errors

    /// @notice Functionality is currently closed
    error Module__ExampleContract__FunctionalityClosed();

    /// @notice Contract or module is already initialized
    error Module__ExampleContract__AlreadyInitialized();

    // ========================================================================
    // Events

    // ------------------------------------------------------------------------
    // State Change Events

    /// @notice Emitted when functionality is enabled
    event FunctionalityEnabled();

    /// @notice Emitted when functionality is disabled
    event FunctionalityDisabled();

    // ------------------------------------------------------------------------
    // Operation Events

    /// @notice Emitted when an amount is processed for a user
    /// @param user_ Address that received the processed amount
    /// @param amount_ Amount processed
    event AmountProcessed(address indexed user_, uint amount_);

    /// @notice Emitted when the fee is updated
    /// @param newFee_ New fee in basis points
    /// @param oldFee_ Previous fee in basis points
    event FeeUpdated(uint newFee_, uint oldFee_);

    // ========================================================================
    // Public Functions - Getter Functions

    // ------------------------------------------------------------------------
    // State Related Getters

    /// @notice Returns whether functionality is open
    /// @return isOpen_ True if open, false if closed
    function isOpen() external view returns (bool isOpen_);

    // ------------------------------------------------------------------------
    // Configuration Getters

    /// @notice Returns the current processing fee
    /// @return fee_ The current fee in basis points
    function getFee() external view returns (uint fee_);

    /// @notice Returns the accepted token address
    /// @return token_ The token address
    function getToken() external view returns (address token_);

    // ========================================================================
    // Public Functions - Mutating Functions

    // ------------------------------------------------------------------------
    // Processing Functions

    /// @notice Process amount for specified receiver
    /// @param receiver_ Address to receive processed tokens
    /// @param amount_ Amount to process
    function processFor(address receiver_, uint amount_) external;

    /// @notice Process amount for caller
    /// @param amount_ Amount to process
    function process(uint amount_) external;

    // ------------------------------------------------------------------------
    // Administrative Functions

    /// @notice Opens the processing functionality
    function openProcessing() external;

    /// @notice Closes the processing functionality
    function closeProcessing() external;

    /// @notice Sets the processing fee
    /// @param fee_ The fee in basis points
    function setFee(uint fee_) external;
}
```

## 6. Contract Inheritance

### Interface Inheritance

Interfaces should contain the same functions as the contract and inherit from any contract that the implementing contract inherits from.

**Hierarchy Best Practices**:
- Interface inheritance should mirror contract inheritance exactly
- Each interface level should only declare functions introduced at that level

**Error and Event Inheritance**:
- Errors and events should be defined in the same interface that introduces related functions
- Inherited interfaces automatically include parent errors and events
- Use consistent error naming: `ContractName__FunctionContext__ErrorType`

**Complex Interface Composition Example**:
```solidity
// Base level - core functionality
interface IBaseInterface_v1 {
    error Module__NotInitialized();
    event ModuleInitialized(address orchestrator);

    function init(bytes memory configData) external;
    function getModuleType() external view returns (string memory);
}

// Complex level - specific implementation
interface IComplexInterface_v1 is IBaseInterface_v1 {
    error Module__ComplexInterface__InvalidAmount();
    event PaymentProcessed(address indexed user_, uint amount_);

    function processPayment(address receiver_, uint amount_) external;
    function getFeeRate() external view returns (uint);
}
```

### Virtual Function Standards

All functions should be marked as `virtual` to enable proper inheritance and future extensibility.

**When to Use Virtual**:
- All public and external functions (unless final implementation)
- All internal functions that may need overriding
- Functions in abstract contracts and base contracts


### Storage Layout Considerations

**Storage Gap Patterns**:
For upgradeable contracts, reserve 50 storage slots to prevent conflicts:

```solidity
contract BaseContract {
    uint internal _value;

    // Reserve 50 slots for future use
    uint[50] private __gap;
}
```

**Note**: Storage gaps are only required for contracts that will be upgraded or extended. Non-upgradeable contracts do not need storage gaps.

### Initialization Patterns

Each contract has an initialization function formatted as `__ContractName_Init()`, similar to OpenZeppelin.

**Top-level contracts** have an `init()` function that calls all underlying init functions in order from lowest to highest level of inheritance.

**Example**:
```solidity
function init(
    IOrchestrator_v1 orchestrator_,
    Metadata memory metadata,
    bytes memory configData
) external override(IModule_v1) initializer {
    // Decode config data
    (address issuanceToken, Properties memory props, address acceptedToken) =
        abi.decode(configData, (address, Properties, address));

    // Deepest level first - later levels may need previous levels data
    __Module_v1_init(orchestrator_, metadata);
    __BaseContract_v1_init();
    __MiddleContract_v1_init();
    __TopContract_v1_init();
}
```

## 7. Documentation Standards

### NatSpec Requirements

**Contract Overview**:
Every contract, interface, and library must have comprehensive NatSpec:

```solidity
/**
 * @title    Contract Name
 *
 * @notice   Brief description of contract functionality
 *
 * @dev      Additional technical information for developers
 *
 * @custom:security-contact security@inverter.network
 *                          In case of concerns, refer to our Security Policy
 *                          at security.inverter.network or email directly!
 *
 * @custom:version   1.1.1
 *
 * @custom:standard-version  1.0.0
 *
 * @author   Inverter Network
 */
```

**Function Documentation**:
- All public/external functions must have NatSpec
- Use `@inheritdoc` when functionality is identical to interface
- Add explicit NatSpec when overriding or extending functionality
- Internal functions should have NatSpec (contract-level since not in interface)

**NatSpec Tag Usage**:
- **`@notice`**: Always required - user-facing description of what the function does
- **`@dev`**: Optional - technical implementation details, security notes, or developer warnings
- **`@param`**: Required for each parameter - describe the parameter's purpose
- **`@return`**: Required for return values - describe what is returned
- **When to use `@dev`**:
  - Security considerations or warnings
  - Implementation details that affect usage
  - Integration notes for other developers
  - Performance considerations

**Examples**:
```solidity
/// @notice The total amount of rewards generated up until now
uint internal _reward;

/// @notice Returns the current reward amount
/// @return amount_ The total amount of rewards available
function getRewards() external view returns (uint amount_);

/// @notice Calculates reward amount at a specific timestamp
/// @dev Should only be called by authorized users due to gas costs
/// @param timestamp_ The timestamp to calculate rewards for
/// @return rewardAmount_ The calculated reward amount at the given time
function _calculateRewardAmount(uint timestamp_) internal view returns (uint rewardAmount_);

/// @notice Maps user addresses to their token balance
/// @dev Key: user address, Value: balance amount in wei
mapping(address user => uint balance) internal _userBalances;

/// @notice Processes a payment with security checks
/// @dev Follows check-effects-interactions pattern for reentrancy protection
/// @param recipient_ Address to receive the payment
/// @param amount_ Amount to transfer in wei
function processPayment(address recipient_, uint amount_) external;
```

**Line Length**: Keep to 80 characters max for comments and NatSpec.

### Folder-based READMEs


Add README files to important folders explaining contents and concepts. Structure them logically according to your project's architecture.

**General Principles**:
- Explain the purpose of each folder
- Document relationships between contracts
- Provide navigation guidance for developers
- Must be updated when contracts are added, removed, or changed

**Example Structures**:

*Smart Contract Folder Structure*:
```
src/
└── contracts/
    └── core/
        ├── Contract1.sol
        └── Contract2.sol
```

*Documentation Structure*:
```
docs/
├── contracts/
│   ├── core/
│   │   ├── Contract1.md
│   │   ├── Contract2.md
│   │   └── README.md
│   └── README.md
└── README.md (links to each contract doc)
```



### Code Comments

**Usage**:
- Complex algorithms need detailed explanations
- Security considerations must be explicitly documented
- Include usage examples for complex functionality

**When to Comment**:
- Non-obvious business logic
- Security-critical sections
- Complex mathematical operations
- Integration points with external contracts

## 8. Testing

### Test Types and Coverage

**Coverage Requirements**:
- Minimum 90% code coverage for all contracts
- 100% coverage for critical functions
- All state transitions must be tested
- Edge cases explicitly tested

**Test Types**:
- **Unit Tests**: Individual function testing (positive/negative paths, fuzz testing)
- **Integration Tests**: Contract interaction testing
- **End-to-End Tests**: Full workflow testing

### Gherkin Test Documentation

All test files must start with Gherkin-style documentation:

```solidity
/* Test _processProtocolFeeViaMinting() function
    ├── Given the fee amount > 0
    │   ├── And the treasury address is invalid
    │   │   └── When the function _processProtocolFeeViaMinting() is called
    │   │       └── Then the transaction should revert
    │   └── And the treasury address is valid
    │       └── When the function _processProtocolFeeViaMinting() is called
    │           └── Then it should mint tokens to treasury
    └── Given the fee amount = 0
        └── When the function _processProtocolFeeViaMinting() is called
            └── Then it should return without minting
*/
```

### Mock Usage

**Access Mocks**: For testing internal functions
- Create `ContractName_v1_Exposed.sol`
- Expose internal functions as `exposed_<functionName>()`
- Example: `exposed_calculateFees()`

**Regular Mocks**: For external dependencies
- Use for non-system-under-test contracts
- Always test against access mocks for the system under test
- Test against regular mocks for external dependencies

### Test Naming Conventions

Follow Gherkin structure in camelCase:
- Format: `test[Visibility][FunctionName]_[outcome]Given[condition]`
- Examples:
  - `testInternalProcessProtocolFeeViaMinting_failsGivenTreasuryAddressInvalid()`
  - `testExternalDeposit_succeedsGivenValidAmountAndApprovedToken()`
  - `testPublicWithdraw_revertsGivenInsufficientBalance()`

**Structure Requirements**:
- Each Gherkin path covered by at least one test
- Tests organized in same order as Gherkin documentation
- Group related tests together

**Error and Event References**:
```solidity
// Events
emit IContract_v1.Contract__EventName(param1, param2);

// Errors
vm.expectRevert(IContract_v1.Contract__ErrorName.selector);
```

## 9. Git Standards

### Branch Strategy

**Branch Structure**:
- **Main Branch**: Production-ready, fully audited, contains mainnet deployments
- **Dev Branch**: Latest changes, internally reviewed, testnet deployments
- **Feature Branches**: Individual features, peer reviewed

**Merge Requirements**:
- Feature → Feature: 1 internal review
- Feature → Dev: 2 internal reviews
- Dev → Main: 2+ internal reviews (preferably different from previous reviewers)

### Commit Messages

For PRs reaching dev or main, use squashed commits with well-formatted messages:

**Format**: `<Keyword>: <Short Description>`

**Keywords**:
- `Feat`: New feature added or extended
- `Fix`: Bug fix or issue resolution
- `Docs`: Documentation changes
- `CI`: CI/CD pipeline changes
- `Refactor`: Code refactoring
- `Test`: Test changes
- `Script`: Script changes
- `Revert`: Reverting existing commit

**Guidelines**:
- Keep title to 72 characters or less
- Use description for additional context if needed
- Avoid including issue numbers, PR numbers, or inappropriate language

### Pull Request Process

**Required Elements**:
- Tests/Changes to Tests
- Scripts/Changes to Scripts
- Updated Documentation

**Review Process**:
- PR maintainers may resolve comments for obvious fixes
- Use GitHub "Suggestion" feature for simple changes
- Tag reviewers for complex discussions
- Don't squash commits until final approval

## 10. Development Workflow

### Development Notes

**TODO Comments**:
- `@todo`: For tasks still to be done
- `@note`: Alternative expression for todos
- Dev and main branches should not contain todos in contracts

**Update Information**:
- `@update-info`: Information for future developers updating code
- Allowed in dev and main branches
- Use for things to keep in mind during updates

### Versioning

**Contract Versioning**:
- Add major version to contract name: `ContractName_v1`
- Add version NatSpec when functions are added/updated:
  ```solidity
  /// @custom:version 1.1.0
  ```
- Update in both function and contract title

**Documentation Versioning**:
- Update contract-level version when any function changes
- Track when functions were added with `@custom:version`

### Deployment Guidelines

**General Principles**:
- All deployments should be deterministic and repeatable
- Use consistent deployment scripts across environments
- Maintain deployment documentation
- Tag releases appropriately

**Environment Separation**:
- Clear distinction between testnet and mainnet deployments
- Environment-specific configuration management
- Proper testing before mainnet deployment

---

