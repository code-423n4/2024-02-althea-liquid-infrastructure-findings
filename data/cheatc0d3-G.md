# Gas Report For Althea Liquid Infrastructure Project

## Summary


|        | Issue                                                   | Instances |
|--------|---------------------------------------------------------|----------:|
| [G-01] | Unnecessary Storage Operations in setThresholds         |          1 |
| [G-02] | _withdrawBalancesTo can be optimized for gas efficiency |          1 |
| [G-03] | Use mappings instead of Loops for gas efficiency        |          1 |
| [G-04] | Inefficient Holder Management                           |          1 |
| [G-05] | Unbounded Loops in `_afterTokenTransfer`                |          1 |
| [G-06] | Block Gas Limit in `withdrawFromManagedNFTs`            |          1 |


## Approach

I conducted a comprehensive review of the smart contract code, focusing on identifying inefficiencies in gas usage. This process entailed scrutinizing each line of code, looking for patterns or practices that might lead to unnecessary gas expenditures. My aim was to pinpoint these areas and suggest refinements that could enhance the contract's gas efficiency, ensuring that it operates more economically without sacrificing its intended capabilities.

## [G-01] Unnecessary Storage Operations in setThresholds

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L117

```solidity

// Clear the thresholds before overwriting
delete thresholdErc20s;
delete thresholdAmounts;

for (uint i = 0; i < newErc20s.length; i++) {
    thresholdErc20s.push(newErc20s[i]);
    thresholdAmounts.push(newAmounts[i]);
}
```

**Mitigation:**

Instead of deleting and repopulating the arrays, you can overwrite the existing entries and adjust the array size as necessary. This approach avoids unnecessary gas consumption on delete and push operations when the new arrays are the same size or smaller than the existing ones.

```solidity

function setThresholds(
    address[] calldata newErc20s,
    uint256[] calldata newAmounts
) public virtual onlyOwnerOrApproved(AccountId) {
    require(
        newErc20s.length == newAmounts.length,
        "threshold values must have the same length"
    );

    // Adjust array sizes first to avoid unnecessary gas costs
    if (newErc20s.length < thresholdErc20s.length) {
        for (uint i = newErc20s.length; i < thresholdErc20s.length; i++) {
            thresholdErc20s.pop();
            thresholdAmounts.pop();
        }
    }

    // Overwrite existing entries and add new ones if necessary
    for (uint i = 0; i < newErc20s.length; i++) {
        if (i < thresholdErc20s.length) {
            thresholdErc20s[i] = newErc20s[i];
            thresholdAmounts[i] = newAmounts[i];
        } else {
            thresholdErc20s.push(newErc20s[i]);
            thresholdAmounts.push(newAmounts[i]);
        }
    }
}
```

## [G-02] _withdrawBalancesTo can be optimized further for gas efficiency

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L170

```solidity

for (uint i = 0; i < erc20s.length; i++) {
    address erc20 = erc20s[i];
    uint256 balance = IERC20(erc20).balanceOf(address(this));
    if (balance > 0) {
        bool result = IERC20(erc20).transfer(destination, balance);
        require(result, "unsuccessful withdrawal");
        amounts[i] = balance;
    }
}
```

**Mitigation:**

The first loop caches the balance of each ERC20 token in the contract. This avoids redundant balanceOf calls, which can be costly due to the SLOAD operation.

```solidity
function _withdrawBalancesTo(
    address[] calldata erc20s,
    address destination
) internal {
    uint256[] memory balances = new uint256[](erc20s.length);
    uint256 totalNonZeroBalances = 0;

    // First pass: cache balances and count non-zero balances
    for (uint i = 0; i < erc20s.length; i++) {
        uint256 balance = IERC20(erc20s[i]).balanceOf(address(this));
        if (balance > 0) {
            balances[i] = balance;
            totalNonZeroBalances++;
        }
    }

    // Allocate memory for non-zero balances and their corresponding ERC20 addresses
    address[] memory nonZeroErc20s = new address[](totalNonZeroBalances);
    uint256[] memory nonZeroBalances = new uint256[](totalNonZeroBalances);
    uint256 index = 0;

    // Second pass: populate arrays with non-zero balances and their corresponding ERC20 addresses
    for (uint i = 0; i < erc20s.length; i++) {
        if (balances[i] > 0) {
            nonZeroErc20s[index] = erc20s[i];
            nonZeroBalances[index] = balances[i];
            index++;
        }
    }

    // Perform the transfers in a separate loop to minimize state changes and checks
    for (uint i = 0; i < totalNonZeroBalances; i++) {
        bool result = IERC20(nonZeroErc20s[i]).transfer(destination, nonZeroBalances[i]);
        require(result, "unsuccessful withdrawal");
    }

    // Emit event after successful withdrawals
    if (totalNonZeroBalances > 0) {
        emit SuccessfulWithdrawal(destination, nonZeroErc20s, nonZeroBalances);
    }
}

```

The first loop caches balances and counts how many are non-zero, while the second loop performs the actual transfers for non-zero balances only. This minimizes the number of condition checks and focuses on processing tokens that actually need to be transferred.

## [G-03] Use mappings instead of Loops for gas efficiency

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L171
 
```solidity
   for (uint i = 0; i < holders.length; i++) {
       if (holders[i] == from) {
           holders[i] = holders[holders.length - 1];
           holders.pop();
       }
   }
```

**Mitigation:** 

Consider optimizing the data structure for managing holders to prevent the need for iteration, such as using a mapping to track indexes in the array.

```solidity
mapping(address => uint256) private holderIndexes;
```

## [G-04] Inefficient Holder Management

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164

The contract inefficiently manages token holders, especially in the `_afterTokenTransfer` function, where it may attempt to remove a holder that does not exist or has already been removed. This can lead to unnecessary gas usage and potential logical errors in distribution.

**Mitigation:**

```solidity
function _afterTokenTransfer(
    address from,
    address to,
    uint256 amount
) internal virtual override {
    if (amount == 0) return;

    if (balanceOf(from) == 0) {
        // Efficiently find and remove the from address from the holders array
        removeHolder(from);
    }

    if (balanceOf(to) == amount) {
        // This was a transfer to a new holder
        holders.push(to);
    }
}

function removeHolder(address holder) private {
    for (uint256 i = 0; i < holders.length; i++) {
        if (holders[i] == holder) {
            holders[i] = holders[holders.length - 1];
            holders.pop();
            break;
        }
    }
}
```

## [G-05] Unbounded Loops in `_afterTokenTransfer`

**LoC:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164

The loop for removing an address from the `holders` array is unbounded, which could lead to gas issues if the array gets too large.

```solidity
function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
            for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }
        }
    }
```

**Mitigation:** 

Consider using a mapping to keep track of active holders to avoid iterating over an array. Alternatively, limit the size of operations or implement gas-efficient data structures such as a linked list.

## [G-06] Block Gas Limit in `withdrawFromManagedNFTs`

**Loc:**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359

Withdrawing from a large number of managed NFTs could exceed the gas limit.

**Mitigation:** 

Adopt a withdrawal pattern where each NFT or a small batch of NFTs can be processed individually or in manageable groups to avoid exceeding the block gas limit.

## Conclusion

This report pinpoints areas for gas optimization, showcasing how strategic adjustments can significantly boost the smart contract's efficiency and lower transaction costs.
