### [NC-1] Checking `erc20EntitlementPerUnit.length` conditional will never hit

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

### [NC-2] Unused functions

**Description:** These two functions `_beforeTokenTransfer` and `_afterTokenTransfer` are only declared, but never used, causing bigger gas costs for deploying the contract.

```javascript
File: contracts/LiquidInfrastructureERC20.sol

/**
     * Implements the lock during distributions, adds `to` to the list of holders when needed
     * @param from token sender
     * @param to  token receiver
     * @param amount  amount sent
     */
    function _beforeTokenTransfer(  <@ Line: 128
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

```javascript
File: contracts/LiquidInfrastructureERC20.sol

 /**
     * Removes `from` from the list of holders when they no longer hold any balance
     * @param from token sender
     * @param to  token receiver
     * @param amount  amount sent
     */
    function _afterTokenTransfer(   <@ Line: 166
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
                }
            }
        }
    }
```

### [NC-3] Inefficient require

**Description:** 

```javascript
File: contracts/LiquidInfrastructureERC20.sol

    /**
     * Transfers a LiquidInfrastructureNFT contract out of the control of this contract
     * @notice LiquidInfrastructureNFTs only hold a single token with a specific id (AccountId), this function
     * only transfers the token with that specific id
     *
     * @param nftContract the NFT to release
     * @param to the new owner of the NFT
     */
    function releaseManagedNFT(     <@ Line: 418
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }

        // By this point the NFT should have been found and removed from ManagedNFTs
@>      require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }

```

**Recommended Mitigation:** 

```diff
    function releaseManagedNFT(    
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }

-       // By this point the NFT should have been found and removed from ManagedNFTs
-       require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }

```