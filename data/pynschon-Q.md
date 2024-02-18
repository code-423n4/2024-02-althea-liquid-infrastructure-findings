# ReleaseManagedNFT will not revert

`LiquidInfrastructureERC20::releaseManagedNFT` is expected to revert when the owner tries to release an NFT that is not managed, but that will not happen due to using `require(true)`, which will always be true.
```solidity
  // By this point the NFT should have been found and removed from ManagedNFTs
  require(true, "unable to find released NFT in ManagedNFTs");
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L430-L431

## Proof of Concept

```solidity
    event ReleaseManagedNFT(address nft, address to);
    function ReleaseMangedNFTDoesNotRevert() external {
        vm.startPrank(_owner);
        vm.expectEmit(address(_infraERC20));
        emit ReleaseManagedNFT(address(_infraERC721Other), address(this));
        _infraERC20.releaseManagedNFT(
            // _infraERC20 owns the nft but it does't manage it
            address(_infraERC721Other),
            address(this)
        );
        vm.stopPrank();
    }
```

## Tools Used

Foundry
