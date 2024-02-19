# Low Risk and Non-Critical Issues

# [01] Possible corruption when mamager calling releaseManagedNFT with a non-exist NFT

The `releaseManagedNFT` logic solely verifies the legitimacy of the owner without conducting any input validation.

If the owner provides a non-valid nftContract, the if condition may not be satisfied, resulting in the failure to release any NFT, While still emitting the `ReleaseManagedNFT` event.

```c
File: LiquidInfrastructureERC20.sol
413:     function releaseManagedNFT(
414:         address nftContract,
415:         address to
416:     ) public onlyOwner nonReentrant {
417:         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
418:         nft.transferFrom(address(this), to, nft.AccountId());
419: 
420:         // Remove the released NFT from the collection
421:         for (uint i = 0; i < ManagedNFTs.length; i++) {
422:             address managed = ManagedNFTs[i];
423:             if (managed == nftContract) {
424:                 // Delete by copying in the last element and then pop the end
425:                 ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
426:                 ManagedNFTs.pop();
427:                 break;
428:             }
429:         }
430:         // By this point the NFT should have been found and removed from ManagedNFTs
431:         require(true, "unable to find released NFT in ManagedNFTs");
432: 
433:         emit ReleaseManagedNFT(nftContract, to);
434:     }

```
