# Repository-Specific Extensions

This document contains detailed, project-specific extensions to the universal Inverter Code Standard. These sections are designed to be copied into individual repository documentation as needed.

## Table of Contents

1. [Smart Contract Module Architecture](#smart-contract-module-architecture)
2. [Module-Specific Naming Conventions](#module-specific-naming-conventions)
3. [Detailed Folder Structure for Workflow Projects](#detailed-folder-structure-for-workflow-projects)
4. [Module Inheritance Patterns](#module-inheritance-patterns)
5. [E2E Test Configuration for Modules](#e2e-test-configuration-for-modules)
6. [Deployment Rules for Multi-Environment Projects](#deployment-rules-for-multi-environment-projects)
7. [Module-Specific CI/CD Configurations](#module-specific-cicd-configurations)

---

## Smart Contract Module Architecture

### Directory Structure for Workflow-Based Projects

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
│   │   └── bondingCurve/       # Bonding curve implementations
│   ├── logicModule/            # Logic Module contracts
│   ├── paymentProcessor/       # Payment Processor module contracts
│   ├── libraries/              # Utility libraries
├── orchestrator/               # Workflow orchestrator
├── factories/                  # Workflow deployment factories
├── proxies/                    # Beacons and proxies for workflow deployment
├── experimental/               # Experimental contracts (unaudited, not for production use)
```

### Test Directory Structure

```
test/
├── e2e/                        # End-to-end integration tests
│   └── [mirrors src/ structure]
├── invariant/                  # Invariant tests
│   └── [mirrors src/ structure]
├── unit/                       # Unit tests
│   └── [mirrors src/ structure with exposed contracts]
├── mocks/                      # Mock contracts
│   ├── internal/               # Mocks for internal contracts
│   │   ├── ModuleFactory_v1_Mock.sol
│   │   ├── Authorizer_v1_Mock.sol
│   │   ├── PaymentProcessor_v1_Mock.sol
│   │   └── ...
│   └── external/               # Mocks for external contracts
│       ├── UMA_Oracle_Mock.sol
│       ├── Sablier_v3_Mock.sol
│       └── ...
```

### Scripts Directory Structure

```
scripts/
├── deploymentScript/           # High-level deployment scripts
│   ├── DeploymentScript.s.sol
│   └── TestnetDeploymentScript.s.sol
├── deploymentSuite/            # Deployment tools and components
│   ├── MetadataCollection_v1.s.sol
│   ├── ModuleBeaconDeployer_v1.s.sol
│   └── ...
└── utils/                      # Utility and maintenance scripts
    └── CreateAndDeployModuleBeacon.s.sol
```

---

## Module-Specific Naming Conventions

### Module Type Abbreviations

Use these abbreviations for module contracts:

**Main Module Types**:
- `LM` - Logic Module
- `AUT` - Authorizer
- `FM` - Funding Manager
- `PP` - Payment Processor

**Module Subtypes**:
- `BC` - Bonding Curve
- `PC` - Payment Client

### Examples

**Funding Manager with Bonding Curve**:
- Contract: `FM_BC_Bancor_Redeeming_VirtualSupply_v1`
- Interface: `IFM_BC_Bancor_Redeeming_VirtualSupply_v1`
- Test: `FM_BC_Bancor_Redeeming_VirtualSupply_v1.t.sol`

**Payment Processor**:
- Contract: `PP_Simple_v1`
- Interface: `IPP_Simple_v1`
- Test: `PP_Simple_v1.t.sol`

### Module Interface Inheritance Pattern

Interfaces should inherit from all base module interfaces:

```solidity
interface IFM_BC_Bancor_Redeeming_VirtualSupply_v1 is
    IVirtualCollateralBase_v1,
    IVirtualSupplyBase_v1,
    IRedeemingBondingCurveBase_v1,
    IBondingCurveBase_v1,
    IFundingManager_v1
{
    // Module-specific functions
}
```

---

## Detailed Folder Structure for Workflow Projects

### Source Code Organization

```
src/
├── external/                   # Peripheral contracts (singletons)
│   ├── governor/
│   │   ├── interfaces/
│   │   │   └── IGovernor_v1.sol
│   │   └── Governor_v1.sol
│   ├── feeManager/
│   └── issuanceToken/
├── factories/
│   ├── interfaces/
│   └── OrchestratorFactory_v1.sol
├── modules/
│   ├── authorizer/
│   │   ├── interfaces/
│   │   ├── extensions/         # If there are extensions
│   │   └── RoleAuthorizer_v1.sol
│   ├── base/
│   │   ├── ModuleBase_v1.sol
│   │   └── interfaces/
│   ├── fundingManager/
│   │   ├── bondingCurve/
│   │   │   ├── abstracts/
│   │   │   │   ├── BondingCurveBase_v1.sol
│   │   │   │   └── RedeemingBondingCurveBase_v1.sol
│   │   │   ├── interfaces/
│   │   │   │   ├── IBondingCurveBase_v1.sol
│   │   │   │   └── README.md (if interfaces need explanation)
│   │   │   ├── FM_BC_Bancor_v1.sol
│   │   │   └── README.md
│   │   ├── simple/
│   │   └── interfaces/
│   ├── logicModule/
│   ├── paymentProcessor/
│   └── libraries/
├── orchestrator/
└── proxies/
```

### Documentation Structure

```
docs/
├── contracts/                  # Mirrors src/ structure
│   ├── modules/
│   │   ├── fundingManager/
│   │   │   ├── bondingCurve/
│   │   │   │   ├── BondingCurveBase_v1.md
│   │   │   │   ├── RedeemingBondingCurveBase_v1.md
│   │   │   │   └── FM_BC_Bancor_v1.md
│   │   │   └── README.md
│   │   └── README.md
│   └── README.md
├── guides/                     # Cross-contract workflows and advanced topics
└── README.md
```

---

## Module Inheritance Patterns

### Initialization Chain Example

Each module follows a specific initialization pattern:

```solidity
function init(
    IOrchestrator_v1 orchestrator_,
    Metadata memory metadata,
    bytes memory configData
) external override(IModule_v1) initializer {
    address _issuanceToken;
    BondingCurveProperties memory bondingCurveProperties;
    address _acceptedToken;

    (_issuanceToken, bondingCurveProperties, _acceptedToken) =
        abi.decode(configData, (address, BondingCurveProperties, address));

    // Deepest level first - later levels may need previous levels data
    __Module_v1_init(orchestrator_, metadata);
    __BondingCurveBase_v1_init();
    __RedeemingBondingCurveBase_v1_init();
    __FM_BC_Bancor_Redeeming_VirtualSupply_v1_init();
}
```

### Base Module Initialization

```solidity
function __Module_init(
    IOrchestrator_v1 orchestrator_,
    Metadata memory metadata
) internal onlyInitializing {
    // Write orchestrator to storage
    if (address(orchestrator_) == address(0)) {
        revert Module__InvalidOrchestratorAddress();
    }
    __Module_orchestrator = orchestrator_;

    // Write metadata to storage
    if (!LibMetadata.isValid(metadata)) {
        revert Module__InvalidMetadata();
    }
    __Module_metadata = metadata;

    emit ModuleInitialized(address(orchestrator_), metadata);
}
```

---

## E2E Test Configuration for Modules

### E2E Test Structure

E2E tests for module-based workflows require special configuration:

```solidity
contract ModuleE2ETest is E2ETest {
    // Module configuration array
    IOrchestratorFactory_v1.ModuleConfig[] moduleConfigurations;

    function setUp() public override {
        super.setUp();

        // Configure modules for workflow
        _setUpModuleConfigurations();

        // Create orchestrator with module configuration
        _createOrchestratorWithModules();
    }

    function _setUpModuleConfigurations() internal {
        // Add Authorizer
        moduleConfigurations.push(
            IOrchestratorFactory_v1.ModuleConfig({
                moduleType: IOrchestratorFactory_v1.ModuleType.Authorizer,
                moduleAddress: address(authorizerBeacon),
                configData: abi.encode(admin, users)
            })
        );

        // Add Funding Manager
        moduleConfigurations.push(
            IOrchestratorFactory_v1.ModuleConfig({
                moduleType: IOrchestratorFactory_v1.ModuleType.FundingManager,
                moduleAddress: address(fundingManagerBeacon),
                configData: abi.encode(token, bondingCurveParams)
            })
        );

        // Add Payment Processor
        moduleConfigurations.push(
            IOrchestratorFactory_v1.ModuleConfig({
                moduleType: IOrchestratorFactory_v1.ModuleType.PaymentProcessor,
                moduleAddress: address(paymentProcessorBeacon),
                configData: abi.encode(token)
            })
        );

        // Add Logic Module
        moduleConfigurations.push(
            IOrchestratorFactory_v1.ModuleConfig({
                moduleType: IOrchestratorFactory_v1.ModuleType.LogicModule,
                moduleAddress: address(logicModuleBeacon),
                configData: abi.encode(params)
            })
        );
    }
}
```

### Module Configuration Testing

```solidity
function testE2E_CompleteWorkflowExecution() public {
    // Test orchestrator initialization
    assertTrue(address(orchestrator) != address(0));

    // Test module initialization
    assertEq(orchestrator.authorizer(), authorizerModule);
    assertEq(orchestrator.fundingManager(), fundingManagerModule);
    assertEq(orchestrator.paymentProcessor(), paymentProcessorModule);
    assertEq(orchestrator.logicModule(), logicModuleModule);

    // Test workflow functionality
    _testCompleteWorkflow();
}
```

---

## Deployment Rules for Multi-Environment Projects

### Environment-Specific Deployments

**Mainnet Deployment (Release)**
- Can always be found on "main" branch
- Each mainnet deployment has a tag (e.g., "v1.0.0")
- Must be fully audited externally
- Requires governance approval for upgrades

**Mainnet Mirror (on Testnet)**
- Same deployment as on mainnet
- Main use:
  - Testing upgrades/changes to mainnet deployment
  - Allowing frontend/SDK to test against mainnet version
  - Example: Regular Control Room users on testnet

**Developer Deployment (Beta)**
- Contains newest changes from "dev" branch
- Automated deployment triggered by specific commit message keywords
- Main use:
  - Testing features before release
  - User testing and QA
  - Beta version of Control Room

### Deployment Automation

**Commit Message Triggers**:
Add specific keywords at the end of commit messages to trigger deployments:
- `[deploy-beta]` - Deploys to testnet beta environment
- `[deploy-staging]` - Deploys to staging environment

**Example**:
```
Feat: Add new bonding curve implementation [deploy-beta]

This commit adds support for linear bonding curves with
configurable slope parameters.
```

---

## Module-Specific CI/CD Configurations

### Module Interface Version Checking

For projects with modular architecture, additional CI checks are needed:

```yaml
# Example CI configuration
interface-compatibility-check:
  runs-on: ubuntu-latest
  steps:
    - name: Check Major Interface Changes
      run: |
        # Check for changes to IAuthorizer, IFundingManager, IPaymentProcessor
        # These require orchestrator updates as well
        if git diff --name-only HEAD~1 | grep -E "(IAuthorizer|IFundingManager|IPaymentProcessor)"; then
          echo "Major interface changed - verify orchestrator compatibility"
          # Run additional checks
        fi

module-inheritance-check:
  runs-on: ubuntu-latest
  steps:
    - name: Check Inheritance Chain Updates
      run: |
        # If BondingCurveBase gets updated, all inherited contracts need updates
        # Verify version tags are updated in inheritance chain

contract-documentation-check:
  runs-on: ubuntu-latest
  steps:
    - name: Verify Contract Documentation
      run: |
        # For each new/modified contract in src/, ensure corresponding .md exists
        # Check for User Flows section
        # Check for Contract/Function Description section
```

### Module-Specific Quality Gates

**Module Version Consistency**:
- When a base module is updated, check all inheriting modules have version bumps
- Verify `@custom:version` tags are updated appropriately
- Check that interface versions match implementation versions

**Module Integration Testing**:
- Run integration tests for all module combinations
- Verify module interfaces are compatible
- Test workflow creation with various module configurations

**Documentation Completeness**:
- Each module must have corresponding documentation
- Module interaction diagrams must be updated
- README files in module folders must reflect current state

---

## Usage Instructions

### For Workflow-Based Projects

Copy the following sections to your project documentation:

1. **Smart Contract Module Architecture** - for projects using the workflow/module pattern
2. **Module-Specific Naming Conventions** - for projects with LM/AUT/FM/PP module types
3. **E2E Test Configuration for Modules** - for testing modular workflows

### For Multi-Environment Projects

Copy the following sections:

1. **Deployment Rules for Multi-Environment Projects** - for projects with mainnet/testnet/beta environments
2. **Module-Specific CI/CD Configurations** - for projects needing interface compatibility checks

### Customization Guidelines

- Replace module type abbreviations with your project's specific types
- Adapt folder structures to match your project's architecture
- Modify deployment environment names to match your setup
- Update interface names to match your project's contracts

### Integration with Universal Standards

These extensions supplement, not replace, the universal standards. Always maintain consistency with:

- Universal naming conventions (add project-specific prefixes/suffixes)
- Universal code layout (extend structure sections as needed)
- Universal testing methodology (add project-specific test patterns)
- Universal documentation requirements (add project-specific sections)

---

## CI/CD and Quality Assurance

### Contract Size Monitoring

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