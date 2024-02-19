# Low [1]

## Title
Incorrect `require` statement in `LiquidInfrastructureERC20::releaseManagedNFT` function

## Description

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431

In the `LiquidInfrastructureERC20::releaseManagedNFT` is used the following `require` statement:

```javascript
require(true, "unable to find released NFT in ManagedNFTs");
```

This statement is incorrect and does not serve any purpose. The `require` statement in Solidity is used to ensure that certain conditions are met. If the condition evaluates to false, the transaction is reverted, and the provided error message is returned. However, since the condition provided here is `true`, the `require` statement will always pass, and the error message will never be used.
It seems intended to check whether a specific `NFT` was successfully found and removed from the `ManagedNFTs` array. The correct implementation should involve a condition that can actually evaluate to `false` if the `NFT` is not found.

## Recommendation

You can add a boolean variable `found` that track whether the `NFT` contract address was found in the `ManagedNFTs` array. If the `NFT` is not found after the loop, the `require` statement will revert the transaction with the provided error message. This ensures that the function only completes successfully if the `NFT` was indeed found and removed from the managed list.

```diff
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

+       bool found = false;

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
+               found = true;
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
+       require(found, "unable to find released NFT in ManagedNFTs");
-       require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }

```
