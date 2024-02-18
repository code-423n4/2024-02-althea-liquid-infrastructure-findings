## [GAS-1] Unnecessary require statement in `LiquidInfrastructureERC20::releaseManagedNFT`
The `require(true, "unable to find released NFT in ManagedNFTs");` statement in the context provided is redundant and has no practical impact on the contract's execution or logic. Since the condition true will always evaluate to true, this require statement will never revert the transaction. It doesn't contribute to error handling or control flow, and removing it would not change the contract's behavior but could slightly reduce gas costs for contract deployment.

```javascript
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
@>      require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```