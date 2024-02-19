### [gas-1] `LiquidInfrastructureERC20::_afterTokenTransfer` functions gas consumption can be lowerd by simplifying state validation.

***Description:*** The function checks the balance of the `from` parameter and sets it inside a `bool` variable and then checks the value data if it true or false.
``` javascript

bool stillHolding = (this.balanceOf(from) != 0); 
        if (!stillHolding) {
            ...
        }

 ```

 But we can make it more simple and gas efficient by using a simple require statement.

 ***Impact:*** Currenly this function validates the data into 2 steps which consumes more gas comparatively if the same can be done with 1 step


***Mitigation:***

We can turn this process into a one step process simply by 

```diff
function _afterTokenTransfer(address from, address to, uint256 amount) internal virtual override {
-  bool stillHolding = (this.balanceOf(from) != 0); 
-    if (!stillHolding) {
+  require(this.balanceOf(from) != 0,"not enough balance") //the function will revert at this point
            for (uint256 i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }
        }
    }

```

### [gas-2] Unnecessary state validation in `LiquidInfrastructureERC20::_beginDistribution` function

***Details:*** The `LiquidInfrastructureERC20::distribute` function checks if `!LockedForDistribution` and then calls `LiquidInfrastructureERC20::_beginDistribution` which again checks for if `!LockedForDistribution` which is totally unnecessary as `LiquidInfrastructureERC20::_beginDistribution` function is only called by `LiquidInfrastructureERC20::distribute` function after checking `!LockedForDistribution` there is no need to check it again.

``` javascript 
function distribute(uint256 numDistributions) public nonReentrant {
        //@audit I have some mixed feelings about this functions
        require(numDistributions > 0, "must process at least 1 distribution");
        if (!LockedForDistribution) {
            require(_isPastMinDistributionPeriod(), "MinDistributionPeriod not met");
            _beginDistribution();
        } 

        uint256 limit = Math.min(nextDistributionRecipient + numDistributions, holders.length);

        ... 
 ```


 ***Impact:*** As these two function is it will double check for  `!LockedForDistribution` and cost gas each time it is checked which is totally gas ineffecient.

***Mitigation:*** Remove `!LockedForDistribution` from `LiquidInfrastructureERC20::_beginDistribution` function as this function is called after checking this bool variable.

``` diff
function _beginDistribution() internal {
-       require(!LockedForDistribution, "cannot begin distribution when already locked");
        LockedForDistribution = true;

        // clear the previous entitlements, if any
        if (erc20EntitlementPerUnit.length > 0) {
            delete erc20EntitlementPerUnit;
        }

```
