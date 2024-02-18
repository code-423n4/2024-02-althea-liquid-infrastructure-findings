# [Low] Blacklisted user prevent distribution() which locks the contract
        

Function `distribute` is used to pay out rewards in LiquidInfrastructureERC20 contract. When distribution started users are prevented from performing transfers, mints, and burns i.e., no token holder is allowed to burn the tokens to claim their tokens / transfer their erc20 tokens to others. This implies that completion of distribution is crucial step.

If one user becomes blacklisted for a token (eg USDC), `distribute()` never gets executed as the step `toDistribute.transfer(recipient, entitlement)` reverts

[LiquidInfrastructureERC20.sol#L220-L227](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L220-L227)

```solidity
  function distribute(uint256 numDistributions) public nonReentrant {
        // ...
        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) { //> @audit DoS: if used blacklisted tokens in distribution  
                        receipts[j] = entitlement;
                    }
                }

                emit Distribution(recipient, distributableERC20s, receipts);
            }
        }
  }
```

However, the owner can change the distribution tokens before distribution, it acts as a safe-guard but also raises trust issues as there is a crucial state change during distribution of contracts which locks the contract

## Impact
Unexpected failures
DoS if owner doesn't handle.

## Recommended Mitigation Steps
1. (May be Unfair) Skip blacklisted users in a processWithdrawals loop.
2. Use a standard token to handle these situations.