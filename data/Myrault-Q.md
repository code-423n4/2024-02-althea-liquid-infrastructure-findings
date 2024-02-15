## There is a meaningless require statement in the `releaseManagedNFT` function of the contract `LiquidInfrastructureERC20.sol`

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
        require(true, "unable to find released NFT in ManagedNFTs"); // <--- here

        emit ReleaseManagedNFT(nftContract, to);
    }
```

This sentence `require(true, "unable to find released NFT in ManagedNFTs");` actually has no effect. The require function is used for condition checking. If the condition is false, an exception will be thrown and the transaction will be rolled back. But the condition here is true, which means that this condition will always be true, so this statement will never trigger an exception.

## Recommended Mitigation Steps
```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..2abf190 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -428,7 +428,6 @@ contract LiquidInfrastructureERC20 is
             }
         }
         // By this point the NFT should have been found and removed from ManagedNFTs
-        require(true, "unable to find released NFT in ManagedNFTs");

         emit ReleaseManagedNFT(nftContract, to);
     }
```