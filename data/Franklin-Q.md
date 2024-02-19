[L-1] **Redundant check in `LiquidInfrastructureERC20::distributeToAllHolders()` and `LiquidInfrastructureERC20:distribute()`**

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