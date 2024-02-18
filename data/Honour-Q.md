## Impact
erc20 distribution can begin before all tokens are sucessfully withdrawn from managed Nft

## Proof of Concept
Both functions `LiquidInfrastructureERC20::distribute` and `LiquidInfrastructureERC20::withdrawFromManagedNfts` lack access controls and while `LiquidInfrastructureERC20::withdrawFromManagedNfts` is pausd/locked during erc20 distribution ,
It is possibe for distribution to begin during batched withdrawals(*i.e multiple transaction calls to `LiquidInfrastructureERC20::withdrawFromManagedNfts` to incrementally withdraw from ManagedNfts*) and cause further attempts at withdrawal to fail. distribution is gas-intensive,has a cooldown and might even require miltiple transactions, all necessary withdrawals should occur before distributing begins  

## Tools Used
Manual review

## Recommended Mitigation Steps
Either use a withdrawal lock to prevent distributions from stating before withdrawals are finished or require that withdrawals be open only during the `minDistributionPeriod`. the former might be a better approach