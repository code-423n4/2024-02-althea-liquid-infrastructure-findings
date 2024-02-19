### Unused `LiquidInfrastructureERC20::_beforeTokenTransfer` and `LiquidInfrastructureERC20::_afterTokenTransfer`  internal function 

***Description:*** Internal functions are the type of functions that can only be called internally within the contract or by the contracts inherited from the current one.
The  `LiquidInfrastructureERC20::_beforeTokenTransfer`and `LiquidInfrastructureERC20::_afterTokenTransfer` function is an `internal virtual override` function but it is not called by any other function which makes this function and its functionality useless.



***Impact:*** If the contract is deployed as it is right now, it will severly impact the usability of the contract causing it to redeploy it after changing it into changing it to a `public` function or calling it from another function.

***Mitigation:*** we can change the function type from `private` to `public` or call it internally via another function.



``` diff
-  function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual override {
+ function _beforeTokenTransfer(address from, address to, uint256 amount) public virtual override {
        
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
            require(isApprovedHolder(to), "receiver not approved to hold the token");
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
or,simply call this function from inside the `LiquidInfrastructureERC20::_beginDistribution` function 
 
 and

 ``` diff
- function _afterTokenTransfer(address from, address to, uint256 amount) internal virtual override {
+ function _afterTokenTransfer(address from, address to, uint256 amount) public virtual override {

        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
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
or,simply call this function from inside the `LiquidInfrastructureERC20::_endDistribution` function