- [L01] invalid require() in function releaseManagedNFT()
  - Need to add some conditions in require() to check whether this is valid release or not.
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

- [L02] Add some check in constructor about _managedNFTs. In Function addManagedNFT(), if we want to add one managedNFT, we have one check to make sure the ownership of NFT. In order to the consistency, we'd better to add similar check in constructor()

```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
        //@johnson,[L] in addManagedNFT, when add one NFT, we need to check ownership, suggest do the same check here
        ManagedNFTs = _managedNFTs;
```

- [L03] Addresses can be pushed into holders even if balanceOf(addr) is zero. For example, Alice sends 0 LiquidInfrastructureERC20 Token to Bob. Although Bob owns 0 Token, Bob addr can still be added in **holders**.

```solidity
    function _beforeTokenTransfer(
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