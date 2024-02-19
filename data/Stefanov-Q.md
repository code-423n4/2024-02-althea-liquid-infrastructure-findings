### [NC] Checking `erc20EntitlementPerUnit.length` conditional will never hit

**Description:** 

After `erc20EntitlementPerUnit` array has been used in `LiquidInfrastructureERC20::distribute`, we call `_endDistribution` which clears the array. This second check in `_beginDistribution` is unnecessary.

```javascript
File: contracts/LiquidInfrastructureERC20.sol

/**
     * Prepares this contract for distribution:
     * - Locks the contract
     * - Calculates the entitlement to protocol-held ERC20s per unit of the LiquidInfrastructureERC20 held
     */
    function _beginDistribution() internal {    <@ Line: 260
        require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );
        LockedForDistribution = true;

        // clear the previous entitlements, if any
@>      if (erc20EntitlementPerUnit.length > 0) {
            delete erc20EntitlementPerUnit;
        }

        // Calculate the entitlement per token held
        uint256 supply = this.totalSupply();
        for (uint i = 0; i < distributableERC20s.length; i++) {
            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );
            uint256 entitlement = balance / supply;
            erc20EntitlementPerUnit.push(entitlement);
        }

        nextDistributionRecipient = 0;
        emit DistributionStarted();
    }
``` 

**Recommended Mitigation:**

```diff
    function _beginDistribution() internal {    
        require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );
        LockedForDistribution = true;

-       // clear the previous entitlements, if any
-       if (erc20EntitlementPerUnit.length > 0) {
-           delete erc20EntitlementPerUnit;
-       }
-
        // Calculate the entitlement per token held
        uint256 supply = this.totalSupply();
        for (uint i = 0; i < distributableERC20s.length; i++) {
            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );
            uint256 entitlement = balance / supply;
            erc20EntitlementPerUnit.push(entitlement);
        }

        nextDistributionRecipient = 0;
        emit DistributionStarted();
    }
```
