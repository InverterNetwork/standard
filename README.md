# Inverter Standard

Version 1.0.0-beta - Last Changed: 2024-12-06

The Inverter Standard is a comprehensive set of guidelines for developing software within the Inverter Network. This standard aims to ensure consistent code quality, security, and maintainability across all projects. For now, it is focused on Solidity smart contracts, but will be expanded to cover all aspects of development in the future.

## Table of Contents
1. [Introduction](#1-introduction)
   - [Purpose](#purpose)
   - [Scope](#scope)
   - [How to Use This Document](#how-to-use-this-document)

2. [General Principles](#2-general-principles)
   - [Code Quality](#code-quality)
   - [Security First](#security-first)
   - [Documentation Requirements](#documentation-requirements)
   - Testing Requirements

3. [Smart Contract Layout](#3-smart-contract-layout)
   - [File Organization](#file-organization)
     - [Directory Structure](#directory-structure)
     - [File Naming](#file-naming)
     - [File Structure](#file-structure)

4. Smart Contract Guidelines

5. [Smart Contract Testing](#5-smart-contract-testing)
   - [Coverage Requirements](#coverage-requirements)
   - [Test Types](#test-types)
   - [Test Naming & Quality](#test-naming--quality)
   - [Test Structure](#test-structure)

6. Documentation Standards

7. Git Standards

8. Development Process

9. Tools and Environment

10. Appendix
    - Templates
    - Checklists
    - Reference Materials

## 1. Introduction

### Purpose
The Inverter Standard serves as the definitive guide for code quality, development practices, and technical processes within the Inverter Network. This standard aims to:
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
   - Follow checklists provided in the appendix for each PR

2. **For Reviewers**
   - Use as a baseline for code review criteria
   - Reference specific sections when providing feedback
   - Ensure all requirements are met before approval

3. **Document Maintenance**
   - Standards are reviewed quarterly
   - Updates require team consensus
   - All changes must be documented in a changelog and tied to a version number
   - Suggestions for improvements are welcome through PRs

4. **Enforcement**
   - All code must comply with these standards
   - Deviations require explicit approval and documentation
   - Automated tools will enforce applicable standards in the future
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
  - Separate concerns into distinct contracts/modules
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
  - All contracts must pass automated security analysis (like Slither, Aderyn, etc.)
  - External contract calls should be treated as potentially malicious
  - State changes must be handled atomically
  - Implement robust access controls
  - Follow check-effects-interactions pattern

- **Known Vulnerabilities**
  - Maintain awareness of common attack vectors
  - Regular security audits and reviews by external parties for any changes
  - Keep updated on latest security developments
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
  - Change logs for all updates
  - Known issues and workarounds
  - Future improvement plans
  - Deprecation notices when applicable

## 3. Smart Contract Standards

### File Organization

#### Directory Structure
```
src/
├── peripheries/                # Contracts not directly included in workflows
│   ├── governor/               # Governor contract
│   ├── feeManager/             # Fee Manager contract
│   ├── issuanceToken/          # ERC20 Issuance Token contract
├── modules/                    # Workflow modules
│   ├── authorizer/             # Authorizer module contracts
│   ├── base/                   # Base contracts for module inheritance
│   ├── fundingManager/         # Funding Manager module contracts
│   ├── logicModule/            # Logic Module contracts
│   ├── paymentProcessor/       # Payment Processor module contracts
│   ├── libraries/              # Utility libraries
├── orchestrator/               # Workflow orchestrator
├── factories/                  # Workflow deployment factories
├── proxies/                    # Beacons and proxies for workflow deployment
├── experimental/               # Experimental contracts (unaudited, not for production use)
```

#### File Naming
- Files should be named using PascalCase, while folders are in camelCase
  - Example: `src/modules/paymentProcessor/PaymentProcessor_v1.sol`
    - Folder: `paymentProcessor` (camelCase)
    - File: `PaymentProcessor_v1.sol` (PascalCase)
- Name match the primary contract within the file
  - Example: `PaymentProcessor_v1.sol` contains `PaymentProcessor_v1`
  - Note: *The name contains the major version number of the contract (in this case v1.x.x) and remainds unchanged if the minor or patch versions change.*
- Interface files are prefixed with "I"
  - Example: `IPaymentProcessor_v1.sol`
- Abstract contracts, also called base contracts, should have the "Base" suffix
  - Example: `BondingCurveBase_v1.sol`

#### File Structure
1. **Licensing**
   ```solidity
   // SPDX-License-Identifier: LGPL-3.0-only
   ```

2. **Pragma Statement**
   - Use specific compiler versions. Contracts use version `0.8.23`, while interfaces use the general version `^0.8.0`.
   ```solidity
   // Contract compiler version 
   pragma solidity 0.8.23;

   // Interface compiler version
   pragma solidity ^0.8.0;
   ```

3. **Import Statements**
   - Group imports by type: external and internal
   ```solidity
   // External
   import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
   
   // Internal
   import "@pp/interfaces/IPaymentProcessor_v1.sol";
   import "@periphery/feeManager/IFeeManager_v1.sol";
   ```

4. **Contract Overview Documentation**
   - Each contract, interface and library have a NatSpec title, containing
     - The name of the contract
     - A description of the contracts functionality
     - Additional information that a developer should know about
     - A security notice
     - The version number of the contract (i.e. v1.0.0)
     - The version number of the Inverter Standard that was used to implement the contract (i.e. v1.0.0)
     - The author of the contract

   Example:
   ```solidity
   /**
    * @title    Inverter Payment Processor
    *
    * @notice   Manages payment orders within a queue and executes payments according
    *           to the configured logic module.
    *
    * @dev      This contract overrides the handler functions of the base contract to
    *           limit users from withdrawing their funds prematurely.
    *
    * @custom:security-contact security@inverter.network
    *                          In case of any concerns or findings, please refer
    *                          to our Security Policy at security.inverter.network
    *                          or email us directly!
    *
    * @custom:version   1.1.1
    *
    * @custom:standard-version  1.0.0
    *
    * @author   Inverter Network
   */
   ```

5. **Contract Declaration**
   - List inheritance from the highest to the lowest level of abstraction
   ```solidity
   contract RedeemingPaymentProcessor_v1 is 
       ITokenRedeemer,
       RedeemingPaymentProcessorBase_v1,
       IPaymentProcessor_v1
   {
       // Contract implementation
   }
   ```

6. **Library Usage**
   ```solidity
   using SafeERC20 for IERC20;
   ```

2. **ERC165 Interface Support**
  - Implement the `supportsInterface()` function, listing all interfaces that the contract implements

  Example:
  ```solidity
  function supportsInterface(bytes4 interfaceId)
        public
        view
        virtual
        override(
            RedeemingPaymentProcessorBase_v1
        )
        returns (bool)
    {
        return interfaceId
            == type(RedeemingPaymentProcessor_v1).interfaceId
            || interfaceId == type(IPaymentProcessor_v1).interfaceId
            || super.supportsInterface(interfaceId);
    }
  ```

8. **Constants**
   ```solidity
   uint256 constant public MAX_DELAY = 30 days;
   bytes32 constant public PROCESSOR_ROLE = keccak256("PROCESSOR_ROLE");
   ```

9. **Storage Variables**
   ```solidity
   // Immutables
   address internal immutable _token;
   
   // State
   mapping(address _user => UserInfo _info) internal _userInfo;
   ProcessorState internal _state;
   ```

10. **Modifiers**
    ```solidity
    modifier userIsAuthorized(address _user) {
        if (!_userInfo[_user].authorized)
            revert Module__UserNotAuthorized(_user);
        _;
    }
    ```

11. **Initialization & Constructor** 
  - Note: *Contracts like modules do not implement a constructor as the base contracts already implement one.*
    ```solidity
    constructor() {
        _disableInitializers();
    }

    function initialize(
        address token_,
        address admin_
    ) external initializer {
        __ModuleBase_init(admin_);
        _token = token_;
    }
    ```

12. **Public Functions**
    ```solidity
    // Getters
    function getProcessorState(uint id_) public virtual view returns (ProcessorState state_) {
        // Implementation
    }
    
    // Mutating functions
    function processPayment(uint256 amount_) external virtual returns (uint256 paymentId_) {
        // Implementation
    }
    ```

13. **Internal Functions**
    ```solidity
    // Regular internal functions
    function _validatePayment(uint256 amount_) internal virtual {
        // Implementation
    }
    
    // Overridden internal functions
    function _handlePaymentTokensAfterProcessing(uint256 amount_) internal virtual override {
        // Implementation
    }
    ```

## 5. Smart Contract Testing
- **Coverage Requirements**
  - Minimum 90% code coverage for all contracts
  - 100% coverage for critical functions and contracts
  - All state transitions must be tested
  - Edge cases must be explicitly tested

- **Test Types**
  - Unit tests for individual functions
    - Cover the positive and negative paths
    - Use fuzz testing where possible
  - Integration tests for contract interactions
    - If external protocols are used, they should be mocked for unit tests and tested via fork testing in integration tests
  - Invariant testing will be introduced in the future and required for critical contracts first

- **Test Naming & Quality**
  - Tests must be readable and maintainable
  - Each test should have a clear purpose
  - Use Gherkin notation for test documentation and naming
  
  #### Gherkin Documentation
  - All test files must start with Gherkin-style documentation that explicitly lists:
    - The requirements of each function
    - All possible execution paths
    - Expected outcomes
  
  Example:
  ```solidity
  /* Test _processProtocolFeeViaMinting() function
      ├── Given the fee amount > 0
      │   └── And the treasury address is invalid
      │       └── When the function _processProtocolFeeViaMinting() is called
      │           └── Then the transaction should revert
  */
  ```

  #### Test Naming Convention
  - Test names should follow the Gherkin structure in camelCase:
    - Format: `test[Visibility][FunctionName]_[outcome]Given[condition]`
    - Visibility: Internal/External/Public
    - Outcome: succeeds/fails/reverts
  
  Examples:
  ```solidity
  // Basic test
  testInternalProcessProtocolFeeViaMinting_failsGivenTreasuryAddressInvalid()
  
  // Multiple conditions
  testExternalDeposit_succeedsGivenValidAmountAndApprovedToken()
  
  // State-dependent test
  testPublicWithdraw_revertsGivenInsufficientBalance()
  ```

  #### Test Structure Requirements
  - Each Gherkin path must be covered by at least one test
  - Tests should be organized in the same order as the Gherkin documentation
  - Complex functions may require multiple tests per path to cover edge cases
  - Group related tests together using describe block