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

## Missing Control Causes DOS
If `MinDistributionPeriod` variable is 0 or 1, users will be able to call function `distribute` any time and that will block owner to `mint` or other users to transfer their tokens. Everyone will be able to call only `distribute`.

```python
>>> liquidierc.MinDistributionPeriod()
1
>>> liquidierc.distribute(4, {'from': bob})
Transaction sent: 0x8a69bd5ee19d2ad09d983608b6b7d8748d0098fbed3663634f7262fc4d1a7720
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 0
  LiquidInfrastructureERC20.distribute confirmed   Block: 14   Gas used: 226029 (1.88%)

<Transaction '0x8a69bd5ee19d2ad09d983608b6b7d8748d0098fbed3663634f7262fc4d1a7720'>
>>> liquidierc.mint(any_acc, 100e18, {'from': deployer})
Transaction sent: 0x8d5b1f3c666a0218c769290cd53002ff583e1fcfecbf3f6becf9cac53373f7d7
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 10
  LiquidInfrastructureERC20.mint confirmed (must distribute before minting or burning)   Block: 15   Gas used: 34346 (0.29%)

<Transaction '0x8d5b1f3c666a0218c769290cd53002ff583e1fcfecbf3f6becf9cac53373f7d7'>
>>> liquidierc.mint(any_acc, 100e18, {'from': deployer})
Transaction sent: 0x7cf3f9f478f834877e8b868c5602c3a97ff0d1a00916909085503034ef20839b
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 11
  LiquidInfrastructureERC20.mint confirmed (must distribute before minting or burning)   Block: 16   Gas used: 34346 (0.29%)

<Transaction '0x7cf3f9f478f834877e8b868c5602c3a97ff0d1a00916909085503034ef20839b'>
>>> liquidierc.mint(any_acc, 100e18, {'from': deployer})
Transaction sent: 0x969652aca32e3de1b376c1394a6af1218a845a99badeeb202d31349e09740af3
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 12
  LiquidInfrastructureERC20.mint confirmed (must distribute before minting or burning)   Block: 17   Gas used: 34346 (0.29%)

<Transaction '0x969652aca32e3de1b376c1394a6af1218a845a99badeeb202d31349e09740af3'>
>>> liquidierc.distribute(4, {'from': bob})
Transaction sent: 0xde231869470f7032fa3d3dbcf47e9cb6140b26a5ce614333090a0fd9fda27157
  Gas price: 0.0 gwei   Gas limit: 12000000   Nonce: 1
  LiquidInfrastructureERC20.distribute confirmed   Block: 18   Gas used: 155829 (1.30%)

<Transaction '0xde231869470f7032fa3d3dbcf47e9cb6140b26a5ce614333090a0fd9fda27157'>
```