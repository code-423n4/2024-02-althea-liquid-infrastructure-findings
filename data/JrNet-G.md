# Remove redundant code in `_beginDistribution()` to save gas

If distribution is called for the first time `erc20EntitlementPerUnit` will be empty and no external/public functions initializations available. And when distribution completed `_endDistribution` ensures to make erc20EntitlementPerUnit empty. So the check can be removed

[LiquidInfrastructureERC20.sol#L265-L267](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L265-L267)

```diff
    function _beginDistribution() internal {
        // ...

        // clear the previous entitlements, if any
-        if (erc20EntitlementPerUnit.length > 0) {
-            delete erc20EntitlementPerUnit;
        }

        // ...
    }
```