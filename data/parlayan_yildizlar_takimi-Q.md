## Released NFT Control Is Missing
On function `releaseManagedNFT`, at the end there should be control to check if given `nftContract` parameter is found in `ManagedNFTs` array or not. However that control is missing. Even if its not found, event will be emitted.

```solidity
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

## Missing Control On Constructor

Function `withdrawFromManagedNFTs` is calling `withdrawFrom.withdrawBalancesTo` and that the function `withdrawBalancesTo` is controlling if msgSender is owner or not.  If it is not it reverts and it will end up reverting whole transaction. So If contract is not the owner of any index in `ManagedNFTs`, whole `withdrawFromManagedNFTs` function will revert. `addManagedNFT` function is controlling if we are the owner or not however on the constructor there is not any control to check if calling contract is owner of each of the given NFT in the array.
```
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
        ManagedNFTs = _managedNFTs;
```