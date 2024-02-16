## There is a part in the require statement that consumes gas and has no actual effect

## Impact
In LiquidInfrastructureERC20.sol, the output of `require(true, "unable to find released NFT in ManagedNFTs")` is consistent under any circumstances. This is a meaningless check that consumes gas.

## Proof of Concept
```
function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs"); 

        emit ReleaseManagedNFT(nftContract, to);
    }
```

### Tools Used
Manual review

## Recommended Mitigation Steps
Directly delete the sentence `require(true, "unable to find released NFT in ManagedNFTs")`