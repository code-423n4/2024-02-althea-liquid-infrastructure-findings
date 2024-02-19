## Althea Liquid Infrastructure GAS OPTIMIZATIONS
## INTRODUCTION
Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime.

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations.

# Gas report

## Table Of Contents

### [G-01] Make Calculations outside of loop to avoid re-calculating on each iteration
**Details**:
Instead of repeatedly calling this.balanceOf(recipient) in the inner loop, you can calculate it once and use the result.

**Proof of Code**
- [LiquidInfrastructureERC20.sol#L198C4-L236C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198C4-L236C10)

```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
214:        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient); //@audit cache outside loop 
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }

                emit Distribution(recipient, distributableERC20s, receipts);
            }
        }
        nextDistributionRecipient = i;

        if (nextDistributionRecipient == holders.length) {
            _endDistribution();
        }
```
**Optimized code:**

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..c519f12 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -212,15 +212,16 @@ contract LiquidInfrastructureERC20 is
         uint i;
         for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
             if (isApprovedHolder(recipient)) {
                 uint256[] memory receipts = new uint256[](
                     distributableERC20s.length
                 );
+                recipientBalance = this.balanceOf(recipient); 
                 for (uint j = 0; j < distributableERC20s.length; j++) {
                     IERC20 toDistribute = IERC20(distributableERC20s[j]);
                     uint256 entitlement = erc20EntitlementPerUnit[j] *
-                        this.balanceOf(recipient);
+                       recipientBalance ; 
                     if (toDistribute.transfer(recipient, entitlement)) {
                         receipts[j] = entitlement;
                     }
```

### [G-02] Do not use `string.concat` if there is only one string use `abi.encodePacked`

**Details**
If there is only one string being concatenated, you can avoid using string.concat and simply concatenate the strings directly. This may slightly reduce gas consumption.
**Proof of Code**
- [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L62)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
62: constructor(
        string memory accountName
    )
        ERC721(
            string.concat(
                "althea://liquid-infrastructure-account/",
                accountName
            ),
            string.concat("LIA:", accountName) //@audit do not use string.concat if there is only one string
        )
    {
        _mint(msg.sender, AccountId);
    }
```
**Optimized code:**

```diff
constructor(string memory accountName)
    ERC721(
-            string.concat(
-                "althea://liquid-infrastructure-account/",
-                accountName
-            ),
-        string.concat("LIA:", accountName)       
+        string(abi.encodePacked("althea://liquid-infrastructure-account/", accountName)),
+        string(abi.encodePacked("LIA:", accountName))
    )
{
    _mint(msg.sender, AccountId);
}
```
### [G-03] Avoid Redundant Calculations instead use cache length

**Details**
Instead of calling holders.length twice, calculate it once and use the result.

**Optimized code:**
- [LiquidInfrastructureERC20.sol#L184C1-L189C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184C1-L189C6)
```diff
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
184:    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
-            distribute(holders.length); //@audit
+            distribute(num);
        }
    }
```
### [G-04] Cache the `.length` outside the loop to avoid multiple recalculations during each iteration.

**Details**
Cache the ManagedNFTs.length outside the loop to avoid multiple recalculations during each iteration.
**Proof of Code**
- [LiquidInfrastructureERC20.sol#L359C5-L385C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C5-L385C10)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
360:  function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        } 
        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length //@audit cache ManagedNFTs.length 
        );
....
        nextWithdrawal = i;

        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```
**Optimized code:**

```diff
 diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..ce52c5d 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -362,10 +363,10 @@ contract LiquidInfrastructureERC20 is
         if (nextWithdrawal == 0) {
             emit WithdrawalStarted();
         }
-
+      uint256 managedNFTsLength = ManagedNFTs.length; 
         uint256 limit = Math.min(
             numWithdrawals + nextWithdrawal,
-            ManagedNFTs.length
+            managedNFTsLength
         );
         uint256 i;
         for (i = nextWithdrawal; i < limit; i++) {
@@ -379,7 +380,7 @@ contract LiquidInfrastructureERC20 is
         }
         nextWithdrawal = i;
 
-        if (nextWithdrawal == ManagedNFTs.length) {
+        if (nextWithdrawal == managedNFTsLength) {
             nextWithdrawal = 0;
             emit WithdrawalFinished();
         }
```
**Details**
Cache the ManagedNFTs.length outside the loop to avoid multiple recalculations during each iteration.
**Proof of Code**
- [LiquidInfrastructureERC20.sol#L412C3-L432C1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L412C3-L432C1)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
414:  function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {`
            address managed = ManagedNFTs[i]; //@audit no need to cache make variable 
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1]; //@audit cache ManagedNFTs.length
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```
**Optimized code:**

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..02c44b9 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -416,13 +417,13 @@ contract LiquidInfrastructureERC20 is
     ) public onlyOwner nonReentrant {
         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
         nft.transferFrom(address(this), to, nft.AccountId());
-
+        uint256 managedNFTsLength = ManagedNFTs.length; 
         // Remove the released NFT from the collection
-        for (uint i = 0; i < ManagedNFTs.length; i++) {
+        for (uint i = 0; i < managedNFTsLength; i++) {
             if (managed == nftContract) {
                 // Delete by copying in the last element and then pop the end
-                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
+                ManagedNFTs[i] = ManagedNFTs[managedNFTsLength - 1];
                 ManagedNFTs.pop();
                 break;
             }
```
**Details**
Cache the newErc20s.length outside the loop to avoid multiple recalculations during each iteration.
**Proof of Code**
- [LiquidInfrastructureNFT.sol#L108C5-L125C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108C5-L125C6)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
108:   function setThresholds(
        address[] calldata newErc20s,
        uint256[] calldata newAmounts
    ) public virtual onlyOwnerOrApproved(AccountId) {
        require(
            newErc20s.length == newAmounts.length,
            "threshold values must have the same length"
        );
        // Clear the thresholds before overwriting
        delete thresholdErc20s;
        delete thresholdAmounts;

        for (uint i = 0; i < newErc20s.length; i++) { //@audit cache length
            thresholdErc20s.push(newErc20s[i]);
            thresholdAmounts.push(newAmounts[i]);
        }
        emit ThresholdsChanged(newErc20s, newAmounts);
    }
```
**Optimized code:**

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol b/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
index 0573e34..f3d7a20 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
@@ -109,15 +109,16 @@ contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
         address[] calldata newErc20s,
         uint256[] calldata newAmounts
     ) public virtual onlyOwnerOrApproved(AccountId) {
+        uint256 newErc20s = newErc20s.length;
         require(
-            newErc20s.length == newAmounts.length,
+            newErc20s == newAmounts.length,
             "threshold values must have the same length"
         );
         // Clear the thresholds before overwriting
         delete thresholdErc20s;
         delete thresholdAmounts;
 
-        for (uint i = 0; i < newErc20s.length; i++) {
+        for (uint i = 0; i < newErc20s; i++) { //@audit cache length
             thresholdErc20s.push(newErc20s[i]);
             thresholdAmounts.push(newAmounts[i]);
         }
```
**Details**
Cache the erc20s.length outside the loop to avoid multiple recalculations during each iteration.
**Proof of Code**
- [LiquidInfrastructureNFT.sol#L170C4-L185C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L170C4-L185C6)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
170: function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) { //@audit 
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```
**Optimized code:**

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol b/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
index 0573e34..7a21269 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
@@ -171,9 +171,10 @@ contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
         address[] calldata erc20s,
         address destination
     ) internal {
+        uint256 erc20s = erc20s.length;
         uint256[] memory amounts = new uint256[](erc20s.length);
-        for (uint i = 0; i < erc20s.length; i++) {
-            address erc20 = erc20s[i];
+        for (uint i = 0; i < erc20s; i++) { //@audit 
+            address erc20 = erc20s[i]; 
             uint256 balance = IERC20(erc20).balanceOf(address(this));
             if (balance > 0) {
                 bool result = IERC20(erc20).transfer(destination, balance);
```
**Details**
Cache the holders.length outside the loop to avoid multiple recalculations during each iteration.
**Proof of Code**
- [LiquidInfrastructureERC20.sol#L164C4-L178C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164C4-L178C10)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
164: function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
            for (uint i = 0; i < holders.length; i++) { //@audit
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }
        }
    }
```
**Optimized code:**

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..bc3bbf4 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -168,10 +168,11 @@ contract LiquidInfrastructureERC20 is
     ) internal virtual override {
         bool stillHolding = (this.balanceOf(from) != 0);
         if (!stillHolding) {
-            for (uint i = 0; i < holders.length; i++) {
+            uint256 holder=  holders.length;
+            for (uint i = 0; i < holder; i++) { 
                 if (holders[i] == from) {
                     // Remove the element at i by copying the last one into its place and removing the last element
-                    holders[i] = holders[holders.length - 1];
+                    holders[i] = holders[holder - 1];
                     holders.pop();
```
### [G-05] Creating variables outside the loop can save few gas

**Details**
creating variables outside the loop can potentially save gas.
**Proof of Code**
- [LiquidInfrastructureNFT.sol#L176](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L176)
```solidity
File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
176: address erc20 = erc20s[i]; //@audit make variable outside loop
```
- [LiquidInfrastructureERC20.sol#L215](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L215)
```solidity
File:
215:     address recipient = holders[i]; //@audit make variable outside loop
```