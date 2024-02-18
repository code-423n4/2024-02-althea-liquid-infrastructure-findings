### [L-1] Unbound array in a for loop in `LiquidInfrastructureERC20::_beginDistribution` && `LiquidInfrastructureERC20::distribute` is a potential denial of service, incrementing gas costs the more managed NFTs there are.

**Description:** The `LiquidInfrastructureERC20::_beginDistribution` && `LiquidInfrastructureERC20::distribute` functions loops through the `distributableERC20s` array to distribute rewards. However, the longer the `LiquidInfrastructureERC20::distributableERC20s` array is, the more gas it will costs. This means that in extreme cases where the protocol have a lot of distributableERC20s it could block the contract from distributing them.

**Impact:** Distribution locked until the owner reduces the size of the array by calling `setDistributableERC20s`.

**Proof of Concept:**
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L220

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L271

**Recommended Mitigation:** Add variables to restrain the maxSize of the distribution and allow it to be made over multiple blocks.

### [L-2] Unbound array in a for loop in `LiquidInfrastructureERC20::constructor` could result in failing the contract deployement.

**Description:** The `LiquidInfrastructureERC20::constructor` function loops through the `_approvedHolders` array to add them to the `HolderAllowlist` array. If the array given in parameters is too big it could prevent the deployement of the contract.

**Impact:** Contract couldnt deploy.

**Proof of Concept:**
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L467

**Recommended Mitigation:** Dont approve a big amount of holders at first, and/or add them after deployement by using the `approveHolder` function.