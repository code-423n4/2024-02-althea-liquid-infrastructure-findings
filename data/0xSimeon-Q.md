## Summary 

| id  | title                 |
|-----|-----------------------|
|  [L-0]   | `LiquidInfrastructureERC20::constructor` does not follow the solidity style guide for functions.|
|  [L-1]   |  Missing zero address check before adding `_approvedHolders` to mapping    |
|  [L-2]   | Irrelevant nSLOC in `LiquidInfrastructureERC20::releaseManagedNFT`       |
| [L-3] | ThresholdERC20s are not accounted for properly.  |


### [L-0] - `LiquidInfrastructureERC20::constructor` does not follow the solidity style guide for functions

[The solidity style guide](https://docs.soliditylang.org/en/latest/style-guide.html#order-of-functions) recommends that constructor should come first in a contract. In `LiquidInfrastructureERC20` (see: [L456](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L456)) the _constructor_ is placed at the bottom. It should be moved to the top. Doing so will also make it consistent with `LiquidInfrastructureNFT` constructor. 

### [L-1] -   Missing zero address check before adding `_approvedHolders` to mapping

There is a missing zero address check in [L467](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L467) which might cause zero address to be included in the `HolderAllowlist` mapping. we should revert early if any zero address is included in passed values. 


### [L-2]  - Irrelevant nSLOC in `LiquidInfrastructureERC20::releaseManagedNFT`

[L431](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431) in `LiquidInfrastructureERC20` will never get triggered since the condition never changes, always true. 

###  [L-3] -  ThresholdERC20s are not accounted for properly.

If an existing holder has `ThresholdERC20s ` in the protocol and the ThresholdERC20s array is updated by calling `setThresholds`, the holder might be unable to withdraw because the previous ERC20s are longer in the ThresholdERC20s array. 