The codebase contains instances in which unnecessary code or code that serve no use  is used 
```
  function approveHolder(address holder) public onlyOwner {
        require(!isApprovedHolder(holder), "holder already approved");
        HolderAllowlist[holder] = true;
    }

      function disapproveHolder(address holder) public onlyOwner {
        require(isApprovedHolder(holder), "holder not approved");
        HolderAllowlist[holder] = false;
    } 

    Both require statements have no effect on the following code and just end up using more gas ie if the owner wants to approve an address, it doesnt matter that the address is already approved, either ways the function logic approves the address
```

```
 function _beginDistribution() internal {
        require(
            !LockedForDistribution, 
            "cannot begin distribution when already locked"
        ); 
        LockedForDistribution = true; 

        // clear the previous entitlements, if any
        if (erc20EntitlementPerUnit.length > 0) {
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

    require(
            !LockedForDistribution, 
            "cannot begin distribution when already locked"
        );  can never fail as it only only called once and only when the require condition is true 
```