# QA Report For Althea Liquid Infrastructure Project

## Summary

|        | Issue                                                    | Instances | 
|--------|----------------------------------------------------------|----------:|
| [L-01] | Broad Use of onlyOwner Without Role-Based Access Control |          8 |
| [L-02] | Precision loss in Distribution Calculation              |          1 |
| [L-03] | Integer Division Leading to Loss of Precision            |          1 |
| [L-04] | DoS in distributeToAllHolders and distribute         |          1 |
| [L-05] | Locking Mechanism during Distribution can be Improved    |          1 |
| [L-06] | Approval and Disapproval of Holders is centralized       |          1 |
| [L-07] | Unchecked Return Values for ERC20 Transfers              |          1 |
| [L-08] | Logic Errors in Distribution Calculation and Execution   |          1 |
| [L-09] | Lack of Input Validation                                 |          1 |
| [L-10] | Missing Upgrade Mechanism                                |          1 |
| [L-11] | Distribution and Withdrawal Front-running risks          |          1 |
| [L-12] | Inaccurate Block Time Dependence for Distribution Periods|          1 |
| [L-13] | Unpredictable Distribution Locking                       |          1 |
| [L-14] | Potential Reentrancy in _beforeTokenTransfer             |          1 |
| [L-14] | Missing Validation on Mint and Burn                      |          1 |
| [L-15] | Using Fixed Solidity Version limits compatibility and upgradability | 2 |
| [L-16] | No Check for Zero Address on Withdrawals                 |          1 |
| [NC-01]| Hardcoded Version Constant                             |          1 |


## Approach

I conducted a detailed QA review of the Althea Liquid Infrastructure Project, focusing on identifying low and non-critical issues in the smart contract code. This review was centered on examining code practices and patterns that could potentially introduce inefficiencies or vulnerabilities, albeit at a lower criticality level. My aim was to uncover areas where enhancements could be made to improve the contract's overall robustness and security.

## [L-01] Broad Use of onlyOwner Without Role-Based Access Control

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L203

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L321

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441

Using onlyOwner for all administrative actions limits the flexibility and decentralization of contract management. It concentrates power in the hands of a single account, which might not be suitable for all operations, especially in a decentralized system aiming for distributed governance.

**Mitigation:**

Implement role-based access control (RBAC) using OpenZeppelin's AccessControl contract. This allows for more granular permissions and the ability to assign specific roles to different tasks, improving security and flexibility. For instance, creating roles like DISTRIBUTOR_ROLE for managing distributions or ASSET_MANAGER_ROLE for handling NFTs can distribute responsibilities more effectively.


## [L-02] Precision loss in Distribution Calculation

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L222C21-L223C51

```
uint256 entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient);
```

**Mitigation:**

Swap the order to avoid precision loss

```
uint256 entitlement = (this.balanceOf(recipient) * erc20EntitlementPerUnit[j]);
```

## [L-03] Integer Division Leading to Loss of Precision

Calculation of entitlement per token might suffer from precision loss due to integer division.

```
uint256 entitlement = balance / supply;
```

**Mitigation:**
 
Use SafeMath or ensure multiplication before division for correct proportion calculation.

```
uint256 entitlement = (balance * 1e18) / supply; // Then, adjust the multiplication factor accordingly when using the entitlement
```

## [L-04] DoS in `distributeToAllHolders` and `distribute`

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184

The `distributeToAllHolders` and `distribute` functions could potentially run into gas limit issues when there are a large number of token holders, thus preventing the completion of distributions.

```solidity
function distributeToAllHolders() public {
    uint256 num = holders.length;
    if (num > 0) {
        distribute(holders.length);
    }
}

function distribute(uint256 numDistributions) public nonReentrant {
    // Omitted for brevity
}
```

**Mitigation:** 

Implement a pull-over-push strategy for distributions to avoid iterating over potentially large arrays, which can lead to DoS issues due to block gas limit. Instead of pushing distributions, allow token holders to pull their rewards.

## [L-05] Locking Mechanism during Distribution can be Improved

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L80

The locking mechanism controlled by the boolean `LockedForDistribution` is toggled by functions that may not necessarily complete successfully, potentially leading to states where the contract is permanently locked or unlocked incorrectly.

**Mitigation:**

Ensure that state-changing operations are fully completed and validated before toggling lock state, and consider adding emergency unlock functionality that can be controlled through a governance process.

```solidity

function emergencyUnlock() public onlyGovernance {
    LockedForDistribution = false;
    emit DistributionFinished(); // Adjust as necessary for correct event signaling
}
```

## [L-06] Approval and Disapproval of Holders is centralized

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116

The functions `approveHolder` and `disapproveHolder` allow the contract owner to unilaterally control who can or cannot hold the token. This centralizes power and can be misused.

**Mitigation:**

Consider implementing a DAO-based voting system for these actions, where token holders can vote on approvals and disapprovals.


## [L-07] Unchecked Return Values for ERC20 Transfers

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224

```
if (toDistribute.transfer(recipient, entitlement)) {
    receipts[j] = entitlement;
}
```

The contract does not check the return value of ERC20 `transfer` calls within the distribution function.

**Mitigation:**

Verify Transfer Success

```solidity
require(toDistribute.transfer(recipient, entitlement), "ERC20 transfer failed");
```

## [L-08] Logic Errors in Distribution Calculation and Execution

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L275

The distribution logic calculates entitlements based on the current token balance and the number of holders at the time of distribution. Changes in holder balances or the addition/removal of holders between distributions could lead to inaccuracies or unintended distribution outcomes.

**Mitigation:**

Implement a snapshot mechanism that records holder balances at the start of each distribution period, ensuring that distributions are based on a consistent set of data.


## [L-09] Lack of Input Validation

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106

The code does not validate inputs in functions like `approveHolder`, `disapproveHolder`, `addManagedNFT`, etc., which could lead to unexpected behavior if invalid addresses like zero address are passed.


```solidity
   function approveHolder(address holder) public onlyOwner {
       require(!isApprovedHolder(holder), "holder already approved");
       HolderAllowlist[holder] = true;
   }
```

**Mitigation:**

Add input validation to ensure addresses are non-zero.

```solidity
require(holder != address(0), "Invalid address");
```

## [L-10] Missing Upgrade Mechanism

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L456

The contract does not implement an upgrade mechanism like those provided by OpenZeppelin's `Upgradeable` contracts, making it hard to fix bugs or improve functionality post-deployment.

**Mitigation:** 

Refactor to use OpenZeppelin's upgradeable contract patterns.

```solidity
   // Example structure, actual implementation will vary
   import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
   import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
   import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

   contract LiquidInfrastructureERC20Upgradeable is Initializable, ERC20Upgradeable, ERC20BurnableUpgradeable, OwnableUpgradeable, UUPSUpgradeable {
       function initialize(string memory name, string memory symbol) public initializer {
           __ERC20_init(name, symbol);
           __Ownable_init();
           __UUPSUpgradeable_init();
       }

       function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
   }
```

## [L-11] Distribution and Withdrawal Front-running risks

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359

```solidity
function distribute(uint256 numDistributions) public nonReentrant {
    // ...
    if (!LockedForDistribution) {
        require(
            _isPastMinDistributionPeriod(),
            "MinDistributionPeriod not met"
        );
        _beginDistribution();
    }
    
}

function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
    require(!LockedForDistribution, "cannot withdraw during distribution");

}
```

**Mitigation:**

Implement a mechanism to shuffle or randomize the order of distributions and withdrawals or use a commit-reveal scheme to mitigate the risk of front-running specific distributions or withdrawals.


## [L-12] Inaccurate Block Time Dependence for Distribution Periods

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L243

The contract relies on block numbers to manage the timing of distributions (`MinDistributionPeriod` and `LastDistribution`). Block times in Ethereum can vary and are not a reliable measure for real-time intervals.

```solidity
function _isPastMinDistributionPeriod() internal view returns (bool) {
    if (totalSupply() == 0 || holders.length == 0) {
        return false;
    }
    return (block.number - LastDistribution) >= MinDistributionPeriod;
}
```

**Mitigation:**

Replace block number-based timing with timestamp-based timing to improve accuracy and predictability of distribution periods.

```solidity

uint256 public LastDistributionTimestamp;

// Use a time-based period instead of block count
uint256 public MinDistributionPeriodInSeconds;

function _isPastMinDistributionPeriod() internal view returns (bool) {
    if (totalSupply() == 0 || holders.length == 0) {
        return false;
    }
    return (block.timestamp - LastDistributionTimestamp) >= MinDistributionPeriodInSeconds;
}

// Adjust the constructor and any relevant parts of the contract to initialize and manage the timestamp-based variables
```

## [L-13] Unpredictable Distribution Locking

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L243

Using block numbers for controlling the lock for distributions (`LockedForDistribution`) can lead to unpredictable locking periods due to the variable block time.

The locking mechanism does not directly depend on block numbers; however, the distribution timing check that enables the locking does.

**Mitigation:**

By moving to a timestamp-based approach for managing distribution periods, the predictability of the locking mechanism will improve as it will rely on real-time intervals rather than block counts.

## [L-14] Potential Reentrancy in _beforeTokenTransfer

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127

The internal token transfer function `_beforeTokenTransfer` does not explicitly protect against reentrancy attacks which could be exploited through callbacks in ERC721 transfers or ERC20 operations.

**Mitigation:**

```solidity
// Include reentrancy protection in sensitive transfer operations
function _beforeTokenTransfer(
    address from,
    address to,
    uint256 amount
) internal virtual override nonReentrant {
    require(!LockedForDistribution, "distribution in progress");
    if (!(to == address(0))) {
        require(
            isApprovedHolder(to),
            "receiver not approved to hold the token"
        );
    }
    if (from == address(0) || to == address(0)) {
        _beforeMintOrBurn();
    }
    bool exists = (this.balanceOf(to) != 0);
    if (!exists) {
        holders.push(to);
    }
}
```

## [L-14] Missing Validation on Mint and Burn

**LoC**

**Mitigation:**

```solidity
function mint(address account, uint256 amount) public onlyOwner nonReentrant {
    require(amount > 0, "Mint amount must be greater than 0");
    _mint(account, amount);
}

function burn(uint256 amount) public override {
    require(amount > 0, "Burn amount must be greater than 0");
    super.burn(amount);
}

function burnFrom(address account, uint256 amount) public override {
    require(amount > 0, "Burn amount must be greater than 0");
    super.burnFrom(account, amount);
}
```


## [L-15] Using Fixed Solidity Version limits compatibility and upgradability 

**LoC**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L2

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L2

```solidity
pragma solidity 0.8.12; // Forces solidity compliance
```

**Mitigation** 

Use a more flexible pragma statement for broader compiler compatibility, especially if you plan to upgrade.

```solidity
pragma solidity ^0.8.12;
```


## [L-16] No Check for Zero Address on Withdrawals

**LoC**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L140

```solidity
address destination = ownerOf(AccountId);
_withdrawBalancesTo(erc20s, destination);
```

**Mitigation:**

```solidity
require(destination != address(0), "ERC20: transfer to the zero address");
```

## [NC-01] Hardcoded `Version` Constant

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L54

The `Version` constant is hardcoded and does not reflect subsequent contract updates or logic changes, potentially misleading users or developers about the contract's actual version.

## Conclusion

The QA review of the Althea Liquid Infrastructure Project has surfaced several low and non-critical issues across the smart contract's design and implementation. While the issues identified are of lower criticality, their resolution represents an opportunity to enhance the contract's quality, reduce potential future risks, and align the project more closely with industry standards and best practices. 