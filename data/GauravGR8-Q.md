## Title

A Getter Function is Needed to Access the `LiquidInfrastructureERC20::holders` Array or its Length.

## Description

The `LiquidInfrastructureERC20::holders` array is a private array of addresses. Currently, no event is assigned, and no getter functions are created to provide the public with information about the holders or the number of holders present.

### Impact

Without an option to retrieve information about the holders or the total count of holders, there is no way to facilitate this requirement.

### Proof of Code

Link to code: [LiquidInfrastructureERC20.sol - Line 48](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48)

### Tools Used

Manual Review

### Recommended Mitigation

To address this issue, consider adding the following code to the `LiquidInfrastructureERC20` contract:

```solidity
function getHoldersArray() external view returns (address[] memory) {
    return holders;
}
```

Alternatively, if the contract owner wishes to maintain the privacy of this data, they can add the onlyOwner modifier:
```solidity
function getHoldersArray() external view onlyOwner returns (address[] memory) {
    return holders;
}
```