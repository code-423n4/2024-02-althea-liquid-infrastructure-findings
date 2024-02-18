## [L-01] releaseManagedNFT() may revert due to out of gas exception if the ManagedNFTs array becomes too long

The `releaseManagedNFT()` function is used to remove a managed NFT from the ManagerNFTs array. If for example, the owner tries to remove the last NFT from the array and it is too long, the transaction may revert due to it hitting the block gas limit. This is because the function loops through the entire array to find the target NFT's index in it.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413
```js
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

If this was to occur, it can be mitigated by removing earlier index entries (because the loop has a break line), then removing the target NFT and then adding the temporarily removed NFT's back. Instead of doing this add a mapping that holds the managed NFTs indexes, and used that to access them in constant time. So `mapping(address => uint) managedNFTIndex`.