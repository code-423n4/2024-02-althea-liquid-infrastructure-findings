## Summary
Pointless check in `LiquidInfrastructureERC20.releaseManagedNFT()`.

## Detail
In the function, the `require` statement will always go through as it's checking a tautology:

```solidity
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

```

The intended `require` usage is probably revert if NFT is not found in the array. However, this doesn't work the intended way as `require(true, ...)` will always be true. Hence the statement is coded incorrectly.