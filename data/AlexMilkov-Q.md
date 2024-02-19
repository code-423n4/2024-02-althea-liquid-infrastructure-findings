# Use modifier onlyOwnerOrApproved(AccountId) in LiquidInfrastructureNft.sol contrant
 in functions withdrawBalances() and withdrawBalancesTo()

Link to repo - https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L135-L162

-Before-
```
function withdrawBalances(address[] calldata erc20s) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }

    
    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        _withdrawBalancesTo(erc20s, destination);
    }
```

-After-
```
function withdrawBalances(address[] calldata erc20s) public virtual onlyOwnerOrApproved(AccountId) {
        
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }

    
    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) public virtual onlyOwnerOrApproved(AccountId) {
      
        _withdrawBalancesTo(erc20s, destination);
    }
```


# Require statement in releaseManagedNFT() function is awlays true before emit event.

Link to repo - https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413-L434 

-Code-
```
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
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```

The solution is to add a bool variable which is set on true when NFT is found and removed from ManagedNFTs, see the code below

 ```
 function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        bool removed;
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                removed = true;
                break;
            }
        }
       
        require(removed, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```