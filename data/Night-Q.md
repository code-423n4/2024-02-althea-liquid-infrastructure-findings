# `_beforeTokenTransfer` and `_afterTokenTransfer` are internal functions but they are not being called from within the contract

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127-L146
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164-L179

## Impact
- The [`_beforeTokenTransfer`](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127C14-L146) function in the `LiquidInfrastructureERC20.sol` contract adds users to the `holders` array but if its marked internal and not being called from within the contract, no one will be able to call it and no users can be added to the `holders` array.
- The [`_afterTokenTransfer`](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164-L179) function  in the `LiquidInfrastructureERC20.sol` contract removes users from the `holders` array but if its marked internal and not being called from within the contract, no one will be able to call it and remove users from the holders array.

## Proof of Concept
```solidity

function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127C14-L146
```solidity

function _afterTokenTransfer(
        address from,
        address to, 
        uint256 amount 
    ) internal virtual override {

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164-L179

## Tools Used
Manual Review