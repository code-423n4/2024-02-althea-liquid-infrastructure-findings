## Function ```releaseManagedNFT``` will not revert and event will be emitted when removing an NFT which is not contained in ```ManagedNFTs```

## Link to affected code
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413-L433

## Details
Since NFTs owned by ```LiquidInfrastructureERC20``` are to be added manually by invoking ```addManagedNFT```, it could happen that the there is an NFT owned by ```LiquidInfrastructureERC20``` but it is not contained in ```ManagedNFTs```. In this case, when ```releaseManagedNFT``` is invoked to remove this NFT which is not contained in ```ManagedNFTs```, the function will not revert and the event ```ReleaseManagedNFT``` will be emitted.
This is classified as low because it is unknown whether it is acceptable to transfer an NFT which is not contained in ```ManagedNFTs```. It is also unknown whether the event emitted will be used either onchain or offchain to trigger some additional action which could malfunction caused by this behavior.

## Tools Used
Manual review

## Recommended Mitigation Steps
It is recommended to check that the NFT is contained within ```ManagedNFTs```:

    function releaseManagedNFT(
            address nftContract,
            address to
        ) public onlyOwner nonReentrant {
            LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
            nft.transferFrom(address(this), to, nft.AccountId());
            bool found;
    
            // Remove the released NFT from the collection
            for (uint i = 0; i < ManagedNFTs.length; i++) {
                address managed = ManagedNFTs[i];
                if (managed == nftContract) {
                    // Delete by copying in the last element and then pop the end
                    ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                    ManagedNFTs.pop();
                    found = true;
                    break;
                }
            }
            // By this point the NFT should have been found and removed from ManagedNFTs
            require(found, "unable to find released NFT in ManagedNFTs");
    
            emit ReleaseManagedNFT(nftContract, to);
        }