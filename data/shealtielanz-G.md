`Gas Optimizations.`
# Report
## Gas Saving Findings 
**Gas Findings List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| G-01 | Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs. | 1 |
| G-02 | Use modifiers instead of repeating checks in different functions. | 3 |
| G-03 | Delete the `mint()` function to save deployment gas | 1 |


# G-01 Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs.
## Summary.
The logic of `addManagedNFT()` allows the owner to only add one token to ManageNFTs in a call:
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394C1-L403C6
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
A call to this function consumes `70119`+ gas on average, In a situation where the owner wishes to add multiple tokens to `ManagedNFTS` it will consume a lot of gas when he calls it multiple times, the function can be reconstructed to add multiple tokens to the `ManagedNFTS` at once.
> [!NOTE]
> The gas price on mainnet isn't always the same so multiple calls to this function might be far more than intended.

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


# G-02 Use modifiers instead of repeating checks in different functions.
## Summary.
In `LiquidInfrastructureERC20` before most functions are called, it checks the value of `LockedForDistribution`, to enforce requirements before certain functions are called, some of those functions are:
- `withdrawFromManagedNFTs()` ~ https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C1-L360C78
- `_endDistribution()` ~ https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L286C1-L290C11
- `_beginDistribution()` ~ https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L257C1-L261C11
- `_beforeTokenTransfer()` ~ https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127C1-L132C69
Instead of constant checks in the function modifiers can be used to check this, 
> [!Tip]
> Since `LockedForDistribution` is a boolean variable, 2 modifiers can be used to do those checks and then appended to the functions that require them appropriately.

Using a modifier saves 40 gas, than the normal require statement in a function.
## Recommendation.
Create a `lockedFalse` and `lockedTrue` modifier that would check the value of the `LockedForDistribution` to guard the functions using.
**Sample of fix**
```solidity
modifier lockedFalse(){
   require(!LockedForDistribution, "distribution in progress");
_;
}

modifier lockedTrue(){
        require(
            LockedForDistribution,
            "cannot end distribution when not locked"
        );
      _;
}
``` 
**Append the `lockedFalse` modifier to:**
- `_beginDistribution()`
- `_beforeTokenTransfer()`
- `withdrawFromManagedNFTs()`

*Append the `lockedTrue` modifier to:**
- `_endDistribution()`



# G-03 Delete the `mint()` function to save deployment gas.
## Summary.
> [!NOTE]
> As is stated in my QA Report.

The `mint()` and `mintAndDistribute()` contain the same logic, except that the `mintAndDistribute()` checks if `_isPastMinDistributionPeriod()` and attempts to distribute if `true`:
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303C1-L323C6
```solidity
    function mintAndDistribute(
        address account,
        uint256 amount
    ) public onlyOwner {
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        mint(account, amount);
    }


  function mint(
        address account,
        uint256 amount
    ) public onlyOwner nonReentrant {
        _mint(account, amount);
    }

```

The thing here is that `mint()` will always revert if the `_isPastMinDistributionPeriod()` returns `true`, so the caller will still have to manually call `distribute()` before he can successfully call `mint()`. this is because of the hooks placed when minting and burning:
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303C1-L311C6
```solidity
   function _beforeMintOrBurn() internal view {
        if 
        require(
            !_isPastMinDistributionPeriod(),
            "must distribute before minting or burning"
        );
    }
```
so it is useless and will cost gas on deployment to have two minting functions when both behave exactly the same.
> [!TIP] 
> Deploying the contract with 2 function that have same restrictions and functionalities under the hood will cost more gas on deployment.
## Recommended Mitigation.
Deployment gas can be saved here by clearing the `mint()` function and sticking with the `mintAndDistribute()` 
**sample of fix**
```diff
Files: LiquidInfrastrucureERC20.sol

    function mintAndDistribute(
        address account,
        uint256 amount
-    ) public onlyOwner {
+    ) public onlyOwner nonReentrant {
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        _mint(account, amount);
    }

-   function mint(
-        address account,
-        uint256 amount
-    ) public onlyOwner nonReentrant {
-       _mint(account, amount);
-   }
```



