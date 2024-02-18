| ID | Description | Severity |
| :-: | - | :-: |
| [L-01](#l-01-supportsinterface-method-in-liquidinfrastructureerc20-is-not-implemented) | `supportsInterface` method in `LiquidInfrastructureERC20` is not implemented. | Low |
| [L-02](#l-02-use-safetransferfrom-instead-of-transferfrom-in-releasemanagednft-function) | Use `safeTransferFrom` instead of `transferFrom` in `releaseManagedNFT` function. | Low |
| [NC-01](#nc-01-withdrawfrommanagednfts-can-be-called-with-zero-value) | `withdrawFromManagedNFTs` can be called with zero value. | Non-critical  
| [NC-02](#nc-02-addmanagednft-function-allows-owner-to-add-same-nft-more-than-once) | `addManagedNFT` function allows owner to add same NFT more than once. | Non-critical 
| [NC-03](#nc-03-calls-to-withdrawbalances-and-withdrawbalancesto-functions-with-empty-array-waste-callers-gas) | Calls to `withdrawBalances` and `withdrawBalancesTo` functions with empty array waste caller's gas. | Non-critical |


# [L-01] `supportsInterface` method in `LiquidInfrastructureERC20` is not implemented.

## Description
According to EIP-165, a contract’s implementation of the `supportsInterface` function should return true for the interfaces that the contract supports. `LiquidInfrastructureERC20` inherited from `ERC721Holder` and implements `onERC721Received` method. However, this contract doesn't have the `supportsInterface` method which show that it implements `ERC721Holder` and the contract can receive an NFT.

## Recommendation
Add `supportsInterface` function to `LiquidInfrastructureERC20` since it inherited from `ERC721Holder`  

# [L-02] Use `safeTransferFrom` instead of `transferFrom` in `releaseManagedNFT` function.

## Description
It is recommended to use `safeTransferFrom` instead of `transferFrom` when transferring ERC721s out of the `LiquidInfrastructureERC20` contract using `releaseManagedNFT` function.
The recipient could have logic in the `onERC721Received` function, which is only triggered in the `safeTransferFrom` function and not in `transferFrom`.

## Recommendation
Consider to use `safeTransferFrom` method.

```diff
function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
--      nft.transferFrom(address(this), to, nft.AccountId());
++      nft.safeTransferFrom(address(this), to, nft.AccountId());

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
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```

# [NC-01] `withdrawFromManagedNFTs` can be called with zero value.

## Description
`withdrawFromManagedNFTs` can be called by any user withh zero as input value. This call does nothing but waste the user's gas.

## Recommendation
Consider to add restriction to call this function with zero value:

```diff
function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");
++      require(numWithdrawals != 0, "Invalid numWithdrawals");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }
        ...
```




# [NC-02] `addManagedNFT` function allows owner to add same NFT more than once.
## Description
An Onwer can push the same NFT contract into `ManagedNFTs` more than once. It does nothing, but waste owner's its gas.

## Recommendation
Consider to add restriction to not allow the owner to add specific NFT more than once.

```diff
function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
++      for (uint i; i < ManagedNFTs.length; i++) {
++          require(ManagedNFTs[i] != nftContract, "Already added");
++      }
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```

# [NC-03] Calls to `withdrawBalances` and `withdrawBalancesTo` functions with empty array waste caller's gas.

## Description
In `LiquidInfrastructureNFT` it's possible to call `withdrawBalancesTo` and `withdrawBalances` functions with empty `erc20s` array. It can lead that user that call these functions waste gas.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L135

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L154

## Recommendation

Add require to `withdrawBalancesTo` function to now allow to withdraw with empty array.

```diff
function withdrawBalancesTo( 
        address[] calldata erc20s,
        address destination
    ) public virtual {
++      require(erc20s.length != 0, "InvalidLength");
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        _withdrawBalancesTo(erc20s, destination);
    }
```

add same check to `withdrawBalances` function.

```diff
function withdrawBalances(address[] calldata erc20s) public virtual { 
    require(
        _isApprovedOrOwner(_msgSender(), AccountId),
        "caller is not the owner of the Account token and is not approved either"
    );
++  require(erc20s.length != 0, "InvalidLength");
    address destination = ownerOf(AccountId);
    _withdrawBalancesTo(erc20s, destination);
}
```