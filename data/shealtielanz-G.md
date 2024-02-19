`Gas Optimizations.`
# Report
## Gas Saving Findings 
**Gas Findings List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| G-01 | Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs. | 1 |
| G-02 | Use modifiers instead of repeating checks in different functions. | 3 |
| G-03 | Delete the `mint()` function to save deployment gas | 1 |
| G-04 | Merge the `burnAndDistribute` & `burnFromAndDistribute` functions using a bool | 1 |


# G-01 Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs.
## Summary.
The logic of `addManagedNFT()` allows the owner to only add one token to ManageNFTs in a call:
```solidity
   function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }

```
A call to this function consumes `70119`+ gas on average, In a situation the owner wishes to add multiple tokens to `ManagedNFTS` it will consume a lot of gas.
> [!NOTE]
> The gas price on mainnet isn't always this same so multiple call to this function might far more than intended.
## Recommendation.
Reconstruct the `addManagedNFT()` to be able to add multiple tokens to the `ManagedNFTs` state variable.
**Sample of fix**
```diff
-        event AddManagedNFT(address nft);
+        event AddManagedNFTs(address[] addrs);
-    function addManagedNFT(address nftContract) public onlyOwner {
+    function addManagedNFT(address[] memory nftContract) public onlyOwner {
-       LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);   
+        uint256 nftLen = nftContract.length;
+        require(nftLen > 0, "abv 0");
+       address[] nfttokens = new address[](nftLen)
+        for(uint256 i; i < nftLen; i++ ){
+       LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract[i]);
+        nftTokens.push(nft);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
-        ManagedNFTs.push(nftContract);
+        ManagedNFTs.push(nftContract[i]);
        }
-      emit AddManagedNFT(nftContract);
+      emit AddManagedNFTs(nfttokens);
    }
```

# G-03 Delete the `mint()` function to save deployment gas.
## Summary.

## Recommendation.


# G-02 Use modifiers instead of repeating checks in different functions.
## Summary.

## Recommendation.


# G-04 Merge the `burnAndDistribute` & `burnFromAndDistribute` functions using a bool
## Summary.

## Recommendation.

