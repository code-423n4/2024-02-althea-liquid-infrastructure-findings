[L-1] **The require statement in `LiquidInfrastructureERC20::releaseManagedNFT` doesn't validate any condition in the function**

**Description** The require statement in the `releaseManagedNFT` function is intended to validate whether an NFT was successfully found and removed from the ManagedNFTs array. However, the condition provided in the require statement

> require(true, "unable to find released NFT in ManagedNFTs");

always evaluates to true, regardless of whether the NFT was found or not. Leading to misleading behavior.

**Recommended Mitigation** If you want to validate that the NFT was found using the require statement, you can follow the mitigation below.

```diff
+       bool found = false;
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
+              found = true;
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
+        require(found, "unable to find released NFT in ManagedNFTs");
-        require(true, "unable to find released NFT in ManagedNFTs");
```

Otherwise remove the require statement entirely because the if statement already ensures that hence the require statement is unnecessary

[L-2] **Redundant check in `LiquidInfrastructureERC20::distributeToAllHolders()` and `LiquidInfrastructureERC20:distribute()`**

**Description** There is a check that requires `numDistribution > 0` in `LiquidInfrastructureERC20:distribute()`. Therefore there is no need to perform this check again in `LiquidInfrastructureERC20::distributeToAllHolders()`. If holders.length in `distributeToAllHolders()` is 0, the require statement in `distribute()` will catch it and the function will revert

**Recommended Mitigation** Remove the check in `distributeToAllHolders()`

```diff
    function distributeToAllHolders() public {
       uint256 num = holders.length;
+      distribute(num);
-      if (num > 0) {
-          distribute(holders.length);
-       }
    }
```

[L-3] **Redundant check in `LiquidInfrastructureERC20::distribute()` and `LiquidInfrastructureERC20:_beginDistribution()`**

**Description** The `LiquidInfrastructureERC20::distribute()` already ensures that distribution only begins when contract is not locked

```javascript
   if (!LockedForDistribution) {
            require(
                _isPastMinDistributionPeriod(),
                "MinDistributionPeriod not met"
            );
            _beginDistribution();
        }
```
So checking this again with the require statement 
`require(!LockedForDistribution,
            "cannot begin distribution when already locked"
        );
` 
in `_beginDistribution()` is redundant 

**Recommended Mitigation** Remove the check in one of the functions

[L-4] In `LiquidInfrastructureERC20::_isPastMinDistributionPeriod()`, returning false when `totalSupply() || holder's length == 0` can be misleading, caller might think that `MinDistributionPeriod` hasn't passed.

**Recommended Mitigation** Use custom error when totalSupply() || holder's length == 0 and only return false when `MinDistributionPeriod` hasn't passed

```diff
+  error zeroHolderOrSupply()
   function _isPastMinDistributionPeriod() internal view returns (bool) {
        // Do not force a distribution with no holders or supply
        if (totalSupply() == 0 || holders.length == 0) {
+           return zeroHolderOrSupply();
_           return false;
        }

        return (block.number - LastDistribution) >= MinDistributionPeriod;
    }
```

[NC-1] Wrong natspec comment in `LiquidInfrastructureERC20::burnFromAndDistribute`

> * Convenience function that allows an approved sender to distribute when necessary and then burn the approved tokens right after

**Recommended Mitigation** Changed comment to approved `spender` not sender




