## Incorrect check for missing Managed NFT
Link:
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431

Issue:
Even if ManagedNFTs is not found it will not fail 

```
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
```

## NFT will miss withdrawal

Link:
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L425

Issue:
In case of deleting Managed NFT, the deleted NFT position is swapped with last entry. So, if `withdrawFromManagedNFTs` was already called for deleted NFT then withdrawFromManagedNFTs will not be called for the swapped NFT since nextWithdrawal>deletedNFTIndex
So swapped NFT call will be made after nextWithdrawal returns to 0
