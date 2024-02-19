# L-1 Unnecessary/Invalid `require` condition

The `releaseManagedNFT` function includes a mechanism for releasing managed NFTs, which involves searching in a loop. 
After finding the required NFT, the loop is terminated (`break;`). 
After the loop, there is a condition ([`require`](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431)) checking the search result.

```solidity
//LiquidInfrastructureERC20.sol
431:        require(true, "unable to find released NFT in ManagedNFTs"); //audit: change true => 'managed == nftContract'
```
This condition contains a hardcoded declaration of `true`, which is unnecessary. 
To maintain the functionality described in the comment, it should be corrected to:
```solidity
//LiquidInfrastructureERC20.sol
431:        require(managed == nftContract, "unable to find released NFT in ManagedNFTs"); //audit
```


# Q-1 Unnecessary Base Contract `ERC20` in `LiquidInfrastructureERC20`
The `LiquidInfrastructureERC20` contract inherits the `ERC20` and `ERC20Burnable` contracts, which also inherit from `ERC20`, making `ERC20` inheritance redundant.

# Q-2 Redundant Function `isApprovedHolder`

The `LiquidInfrastructureERC20.isApprovedHolder` function merely returns the content of a public mapping. 
Solidity automatically generates getters for all public state variables.

```solidity
    function isApprovedHolder(address account) public view returns (bool) {
        return HolderAllowlist[account];
    }
```
It is then used in:
```solidity
require(!isApprovedHolder(holder), "holder already approved");
```
which can be replaced with:
```solidity
require(!this.HolderAllowlist(holder), "holder already approved");
```
or:
```solidity
require(!HolderAllowlist[holder], "holder already approved");
```

# Q-3 Simplify Function `LiquidInfrastructureERC20._beforeTokenTransfer` for Improved Clarity

The function is invoked before each transfer, including minting/burning, and contains multiple conditions. 
The entire block can be optimized for gas usage and readability.

Before:
```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
            require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
    }
```
After:
```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (to != address(0)) {
            require(isApprovedHolder(to), "receiver not approved to hold the token");
            if (balanceOf(to) != 0) {
                holders.push(to);
            }
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
    }
```
Diff:
```diff
diff --git a/original.md b/modified.md
index 4b80a9f..603489a 100644
--- a/original.md
+++ b/modified.md
@@ -4,17 +4,13 @@
         uint256 amount
     ) internal virtual override {
         require(!LockedForDistribution, "distribution in progress");
-        if (!(to == address(0))) {
-            require(
-                isApprovedHolder(to),
-                "receiver not approved to hold the token"
-            );
+        if (to != address(0)) {
+            require(isApprovedHolder(to), "receiver not approved to hold the token");
+            if (balanceOf(to) != 0) {
+                holders.push(to);
+            }
         }
         if (from == address(0) || to == address(0)) {
             _beforeMintOrBurn();
         }
-        bool exists = (this.balanceOf(to) != 0);
-        if (!exists) {
-            holders.push(to);
-        }
     }
```

# Q-4 Optimize Function `LiquidInfrastructureERC20._afterTokenTransfer`

The function is executed after each transfer, checking if the sender still holds shares, and if not, removes it from the array. 
The check is performed in a loop, which can be terminated upon finding a positive search result.

```diff
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
            for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
+                   break;
                }
            }
        }
    }
```

# Q-5 Misleading Function `LiquidInfrastructureERC20._isPastMinDistributionPeriod`

The function name suggests it checks whether the minimum time between distributions has passed, but it also checks if the contract has holders, which can be misleading. 
If maintaining the functionality, the function name should be changed to something appropriate, e.g., `_isDistributionPossible()`.

# Q-6 Simplify Function `LiquidInfrastructureERC20.mintAndDistribute`

The function uses the public `mint` function, which also includes the `onlyOwner` modifier. 
As `mint` and `_mint` are functionally identical, the lighter version can be used in `mintAndDistribute`.

```diff
diff --git a/original.md b/modified.md
index 311c384..d0e0b1e 100644
--- a/original.md
+++ b/modified.md
@@ -1,9 +1,9 @@
     function mintAndDistribute(
         address account,
         uint256 amount
-    ) public onlyOwner {
+    ) public onlyOwner nonReentrant{
         if (_isPastMinDistributionPeriod()) {
             distributeToAllHolders();
         }
-        mint(account, amount);
+        _mint(account, amount);
     }
```

# Q-7 Optimize Function `LiquidInfrastructureERC20.withdrawFromManagedNFTs`

```diff
diff --git a/original.md b/modified.md
index aa58a8b..e0392c6 100644
--- a/original.md
+++ b/modified.md
@@ -9,8 +9,7 @@
             numWithdrawals + nextWithdrawal,
             ManagedNFTs.length
         );
-        uint256 i;
-        for (i = nextWithdrawal; i < limit; i++) {
+        for (uint256 i

 = nextWithdrawal; i < limit; i++) {
             LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                 ManagedNFTs[i]
             );
@@ -19,10 +18,10 @@
             withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
             emit Withdrawal(address(withdrawFrom));
         }
-        nextWithdrawal = i;
-
-        if (nextWithdrawal == ManagedNFTs.length) {
+        if (i == ManagedNFTs.length) {
             nextWithdrawal = 0;
             emit WithdrawalFinished();
+        }else{
+            nextWithdrawal = i;
         }
     }
```