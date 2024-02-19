# Low

### Tautological assertion is not working as intended

In [LiquidInfrastructureERC20](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431), the requirement will never revert, thus the assertion becomes useless.

```solidity
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
```

If the idea is to revert the function when the managed NFT is not found, consider changing the operation inside the requirement into something which guarantees the loop called break. For example:

```solidity
        require(i != ManagedNFTs.length, "unable to find released NFT in ManagedNFTs");
```

To explain the comparison above, the incremental variable can only reach the length of managed NFTs if the loop ends, meaning no managed NFTs were found.

# Non-Critical/Informational

### Two setting functions can be changed into a single toggle function.

Instead of having a turn on and a turn off function, they can be both reduced into a single function that modifies state on what is input.

```solidity
    function approveHolder(address holder) public onlyOwner {
        require(!isApprovedHolder(holder), "holder already approved");
        HolderAllowlist[holder] = true;
    }

    /**
     * Marks `holder` as NOT approved to hold the token, preventing them from receiving any more of the underlying ERC20.
     *
     * @param holder the account to add to the allowlist
     */
    function disapproveHolder(address holder) public onlyOwner {
        require(isApprovedHolder(holder), "holder not approved");
        HolderAllowlist[holder] = false;
    }
```

### State changing functions missing events

The two above functions should emit events.

### Consider changing a requirement into a modifier

In [LiquidInfrastructureNFT.sol#L157-L160](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L157-L160), consider changing the requirement into a modifier.

### `mintAndDistribute()` should call _mint() instead of mint()

In [LiquidInfrastructureERC20](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303-L311), the function is calling the public function instead of the internal function to mint. The only difference is the nonReentrant modifier, however given no external calls or transfers are called in the mint function nor its modifiers, consider calling `_mint()` directly.