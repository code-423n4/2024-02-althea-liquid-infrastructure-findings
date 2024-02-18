## [L-01] releaseManagedNFT() may revert due to out of gas exception if the ManagedNFTs array becomes too long

The `releaseManagedNFT()` function is used to remove a managed NFT fromm the ManagerNFTs array. This is because it loops through the 
entire array to find the target NFT's index in it.

```js
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

If this was to occur, it can be mitigated by removing earlier index entries (because the loop has a break line), then removing the target NFT and then adding the temporarily removed NFT's back. Instead of doing this add a mapping that holds the managed NFTs indexes, and used that to access them in constant time. So `mapping(address => uint) managedNFTIndex`.

## [L-2] _mint() function flow always iterates through holders array, causing very high gas costs and if the holders array grows too large it will DoS the mint() function permanently and burn() and _afterTokenTransfer() functions will revert in some cases

If the holders array in LiquidInfrastructure.sol becomes too big, the `_afterTokenTransfer()` function will hit the block gas limit and revert on a token transfer where the sender sends all of their tokens or burns all of their tokens. This will also cause the `mint()` function to revert due to it hitting the block gas limit, because it always triggers the `_afterTokenTransfer()` function with `address(0)` and an amount of 0. There is no way to artificially reduce the size of the array, so the mint function will be permanently dossed.

Over time, as more users are approved and the protocol grows, the size of the holders array will grow naturally, increasing gas cost on every mint and ultimately causing transactions to hit the block gas limit.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164
```js
    function _afterTokenTransfer(
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

One way to mitigate would be checking if the sending account is address(0) and skipping the iteration entirely. This will conserve ERC20 functionality, however it is still very much gas intesive. Perhaps a better solution would be to keep the indexes of holders in the holders array in a separate mapping say `holdersIndexes`, which would be of type `address => uint`. This will make access of the index of a holder constant time and will completely prevent this DoS from occuring.

As we can see here, the mint function will always trigger the _afterTokenTransfer with address(0) and amount 0.

https://github.com/OpenZeppelin/openzeppelin-contracts/blob/6edb6dd1ca43d05a762d84c688116b3327f5e490/contracts/token/ERC20/ERC20.sol#L251
```js
    function _mint(address account, uint256 amount) internal virtual {
        require(account != address(0), "ERC20: mint to the zero address");

        _beforeTokenTransfer(address(0), account, amount);

        _totalSupply += amount;
        _balances[account] += amount;
        emit Transfer(address(0), account, amount);

        _afterTokenTransfer(address(0), account, amount);
    }
```
Note: The above link is to the commit of that contract in the openzeppelin repo, as it is a 3 years old I have to reference it this way.. It is the correct version 4.3.1, same as in package.json. You can of course check it locally.