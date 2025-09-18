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

3. [Naming Conventions](#3-naming-conventions)
   - [Contracts](#contracts)
   - [Functions](#functions)
   - [Variables](#variables)
   - [Files and Folders](#files-and-folders)

4. [Code Layout](#4-code-layout)
   - [Contract Structure](#contract-structure)
   - [Interface Structure](#interface-structure)
   - [Script Structure](#script-structure)
   - [Test Structure](#test-structure)

5. [Contract Inheritance](#5-contract-inheritance)
   - [Interface Inheritance](#interface-inheritance)
   - [Initialization Patterns](#initialization-patterns)

6. [Documentation Standards](#6-documentation-standards)
   - [NatSpec Requirements](#natspec-requirements)
   - [Folder-based READMEs](#folder-based-readmes)
   - [Code Comments](#code-comments)

7. [Testing](#7-testing)
   - [Test Types and Coverage](#test-types-and-coverage)
   - [Gherkin Test Documentation](#gherkin-test-documentation)
   - [Mock Usage](#mock-usage)
   - [Test Naming Conventions](#test-naming-conventions)

8. [Git Standards](#8-git-standards)
   - [Branch Strategy](#branch-strategy)
   - [Commit Messages](#commit-messages)
   - [Pull Request Process](#pull-request-process)

9. [CI/CD and Quality Assurance](#9-cicd-and-quality-assurance)
   - [Contract Size Monitoring](#contract-size-monitoring)
   - [Test Coverage Enforcement](#test-coverage-enforcement)
   - [Automated Quality Gates](#automated-quality-gates)

10. [Development Workflow](#10-development-workflow)
    - [Development Notes](#development-notes)
    - [Versioning](#versioning)
    - [Deployment Guidelines](#deployment-guidelines)

11. [Tools and Environment](#11-tools-and-environment)
    - [Required Tools](#required-tools)
    - [Configuration](#configuration)

12. [Best Practices](#12-best-practices)
    - [Contract Size Optimization](#contract-size-optimization)
    - [Security Considerations](#security-considerations)

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
   - Follow checklists provided for each PR @todo ?

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
  - All contracts must pass automated security analysis (like Slither, Aderyn, etc.) @todo ?
  - External contract calls should be treated as potentially malicious @todo ?
  - Implement robust access controls
  - Follow check-effects-interactions pattern

- **Known Vulnerabilities**
  - Maintain awareness of common attack vectors @todo ?
  - Regular security audits and reviews by external parties for any changes
  - Keep updated on latest security developments @todo ?
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

## 3. Naming Conventions

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
- Applies to functions, events, and errors

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

**Mappings**
- Always name mapping parameters
- Example: `mapping(uint id => Bounty bounty) internal _idToBounty`

### Files and Folders

**File Naming**
- Files use PascalCase: `PaymentProcessor_v1.sol`
- Folders use camelCase: `paymentProcessor/`
- File names should match primary contract within
- Interfaces prefixed with "I": `IPaymentProcessor_v1.sol`

## 4. Code Layout

### Contract Structure

All contracts follow this order:

1. **Licensing**: `// SPDX-License-Identifier: LGPL-3.0-only`
2. **Pragma**: `pragma solidity 0.8.23;` (contracts) or `pragma solidity ^0.8.0;` (interfaces)
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

@todo Provide examples maybe in different folder and file?

### Interface Structure

1. **Licensing**: `// SPDX-License-Identifier: LGPL-3.0-only`
2. **Pragma**: `pragma solidity ^0.8.0;`
3. **Imports** (External, then Internal)
4. **Contract Overview Documentation**
5. **Interface Declaration with Inheritance**
6. **Structs**
7. **Events**
8. **Errors**
9. **Functions** (getter, then mutating, sorted by functionality)

### Test Structure

**Unit Tests**:
1. **Licensing**: `// SPDX-License-Identifier: UNLICENSED`
2. **Pragma**: `pragma solidity ^0.8.0;`
3. **Imports** (Internal, External, Tests and Mocks, System under Testing)
4. **Contract Overview Documentation**
5. **Contract Declaration**
6. **Constants**
7. **State**
8. **Setup**
9. **Test Init**
10. **Test Modifier**
11. **Test External** (public + external)
12. **Test Internal** (via exposed_ functions)
13. **Helper Functions**

**Style Guidelines**:
- Top headings distinguished by: `// =========================================`
- Sub-headings by: `// -----------------------------------------`
- Lines always 80 characters total (including padding)

## 5. Contract Inheritance

### Interface Inheritance

Interfaces should contain the same functions as the contract and inherit from any contract that the implementing contract inherits from.

@todo Extend on this for more details

**Example**:
```solidity
interface IComplexContract_v1 is IBaseContract_v1, IMiddleContract_v1 {
    // IComplexContract_v1 specific functions
}
```

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

## 6. Documentation Standards

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

@todo do mapping exampls

**Examples**:
```solidity
/// @notice The amount of rewards generated up until now
uint internal _reward;

/// @notice Returns the reward amount
/// @return amount_ The amount of rewards
function getRewards() external returns (uint amount_)

/// @notice Calculates rewards at this timestamp
/// @dev Shouldn't be called by non-admin
/// @param timestamp_ The current timestamp
/// @return rewardAmount_ The amount of rewards at this moment
function _calculateRewardAmount(uint timestamp_) internal returns (uint rewardAmount_)
```

**Line Length**: Keep to 80 characters max for comments and NatSpec.

### Folder-based READMEs

@todo this doesnt fit

Add README files to important folders explaining contents and concepts. Structure them logically according to your project's architecture.

**General Principles**:
- Explain the purpose of each folder
- Document relationships between contracts
- Provide navigation guidance for developers
- Must be updated when contracts are added, removed, or changed

**Example Structure**:
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

## 7. Testing

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
    │   └── And the treasury address is invalid
    │       └── When the function _processProtocolFeeViaMinting() is called
    │           └── Then the transaction should revert
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

## 8. Git Standards

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

## 9. CI/CD and Quality Assurance

### Contract Size Monitoring @todo ?

- **Hard Warning**: If over the size limit
- **Warning**: If size reaches 90% of limit
- Automated checking in CI pipeline

### Test Coverage Enforcement

- **Hard Limit**: 90% minimum coverage
- **Warning**: If coverage decreases
- Coverage reports generated automatically

### Automated Quality Gates

**Compiler Warnings**: Enforce 0 warnings

**Documentation Checks**:
- Detect missing NatSpec
- Verify function documentation completeness
- Check for version tag updates when functions change

**Code Analysis**:
- Run Slither for security analysis
- Check naming convention compliance
- Detect inline bookmarks (@todo, @note)
- Interface version compatibility checks

**Invariant Testing**: Should always run but not block merging

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

## 11. Tools and Environment @todo Add more? / Move to top section?

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
- Supermaven or similar for code completion

### Configuration

**VSCode Settings**: Repository should include `.vscode` folder with:
- Recommended extensions
- Workspace settings for consistent formatting
- Debug configurations

**Foundry Configuration**: Standardized `foundry.toml` settings

## 12. Best Practices

### Contract Size Optimization

**Modifier Patterns**:
```solidity
modifier userIsAuthorized() {
    _ensureUserIsAuthorized();
    _;
}

function _ensureUserIsAuthorized() internal {
    require(authorized, "User not authorized");
}
```

**General Rules**:
- Move reusable logic to libraries
- Externalize repeated code fragments into functions (DRY principle)
- Consider inlining functions called only once
- Minimize buffer variables (balance with gas efficiency)
- Use optimizer settings appropriately

**Note**: Contract size optimization often conflicts with code readability - decide case-by-case.

### Security Considerations

**Virtual Functions**: All functions should be `virtual` for easy overriding in subsequent contracts

**Access Control**: Use consistent patterns for authorization and authentication

**External Calls**: Always treat external contract calls as potentially malicious

**State Changes**: Handle atomically following check-effects-interactions pattern

**Upgradability**: Consider upgrade patterns early in design

---

