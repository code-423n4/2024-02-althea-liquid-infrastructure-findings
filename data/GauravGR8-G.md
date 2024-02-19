(*Note: These are the continuation of the QA Report. It contained the first Low finding and below are the rest.*)

## [L-2] A Function is Needed to Update the Value of `LiquidInfrastructureERC20::MinDistributionPeriod`.

## Description

The `LiquidInfrastructureERC20::MinDistributionPeriod` determines the minimum period required to start the distribution of tokens again. However, if there is a need to adjust the distribution period in the near future, there is currently no way to do so. Therefore, for safety reasons, a function should be created to update the `MinDistributionPeriod`.

### Proof of Code

Link to code: [LiquidInfrastructureERC20.sol - Line 75](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L75)

### Tools Used

Manual Review

### Recommended Mitigation

To address this issue, consider adding the following code to the `LiquidInfrastructureERC20` contract:

```solidity
function updateMinDistributionPeriod(uint256 _newPeriod) external onlyOwner {
    MinDistributionPeriod = _newPeriod;
}
```

## [L-3] Zero Value Checks for Function Parameters

## Description

There are many functions within the `LiquidInfrastructureERC20` and `LiquidInfrastructureNFT` contracts that update the state variables or undergo certain executions based on the parameters provided. We need to put zero value checks for all these parameters to avoid unnecessary gas consumption.

### Impact
These are not only the case of gas consumptions, but even some functions updates the state variables, and updating a few of these state variables can lead to loss of tokens and balances.

### Instances

- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L96
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303-L306
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L318-L321
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L331
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L344
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441
- https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108-L111

### Tools Used

Manual Review

### Recommended Mitigation

Add `require()` statements to check that no zero values are provided even by mistake, and revert if found so.

## [L-4] Improper Implementation of `LiquidInfrastructureERC20::_beforeMintOrBurn`

## Description

As the names spell, the `LiquidInfrastructureERC20::_beforeMintOrBurn` function expects to mint or burn tokens, but there's no such logic implemented within this function.

### Impact
As per the Openzeppelin's ERC20 contract the `LiquidInfrastructureERC20::_beforeTokenTransfer` 
```
  * - when `from` is zero, `amount` tokens will be minted for `to`.
  * - when `to` is zero, `amount` of ``from``'s tokens will be burned.
```
But there's no such implementation within the contract. Neither `_beforeMintOrBurn` facilitates this logic.

### Tools Used

Manual Review

### Recommended Mitigation

The `LiquidInfrastructureERC20::_beforeMintOrBurn` function should included with the mint and burn logics to execute those functions based on the address of 'to' and 'from' provided. Or add the virtual keyword to the function so that it can be modified further in its inheritance contracts.