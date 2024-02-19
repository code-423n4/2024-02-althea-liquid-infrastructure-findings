# Pausing Withdrawals on Distribution
## Impact
erc20 distribution can begin before all tokens are sucessfully withdrawn from managed Nft

## Proof of Concept
Both functions `LiquidInfrastructureERC20::distribute` and `LiquidInfrastructureERC20::withdrawFromManagedNfts` lack access controls and while `LiquidInfrastructureERC20::withdrawFromManagedNfts` is pausd/locked during erc20 distribution ,
It is possibe for distribution to begin during batched withdrawals(*i.e multiple transaction calls to `LiquidInfrastructureERC20::withdrawFromManagedNfts` to incrementally withdraw from ManagedNfts*) and cause further attempts at withdrawal to fail. distribution is gas-intensive,has a cooldown and might even require miltiple transactions, all necessary withdrawals should occur before distributing begins  

## Recommended Mitigation Steps
Either use a withdrawal lock to prevent distributions from stating before withdrawals are finished or require that withdrawals be open only during the `minDistributionPeriod`. the former might be a better approach



# Circumventing _afterTokenTransfer() loop can impact distribution

## Description
The for-loop in `_afterTokenTransfer()` that removes holders from the holder array when they're no longer holding can use up a lot of gas depending on the length of the array and the holders position. Holders may attempt to circumvent this by not totally transferring tokens(leaving a negligible amount e.g. 1wei), however this means that token distribution will become less and less efficient the more this is done because entitlements will still be calculated even though it might be a really small amount or possibly even zero and a ton of gas will be wasted on distribution

## Recommendations
Track holders position in the array with a mapping 

```sol
File:contracts/LiquidInfrastructureERC20

//@ _beforeTokenTransfer()

143 if (!exists && to != address(0)) {
@>	holderPosition[to] = holders.length;
144     holders.push(to);
145 }

//@ _afterTokenTransfer()

if(!stillHolding && from != address(0)) {
	uint fromPosition = holderPosition[from];
	address last = holders[holders.length-1];
	holders[fromPosition] = last; //put the last holder in current holder position
	holderPosition[last] = fromPosition; // update the last holder position
	holders.pop();
}
```