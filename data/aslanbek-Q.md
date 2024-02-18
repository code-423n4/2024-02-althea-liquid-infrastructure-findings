# L-01 Zero-transfer still adds the recipient to the holders list

`_beforeTokenTransfer` and `_afterTokenTransfer` have the following discrepancy:

1. In `_afterTokenTransfer` if the sender has no balance, they are removed from the holders list.

2. However, if there's a transfer of 0 tokens and the recipient has 0 tokens, in `_beforeTokenTransfer` the recipient will still be added to the list of holders.

Consider adding a check for zero-transfer, so the recipient of zero-transfer will not be added to the list of holders.

# L-02 `mintAndDistribute`, `burnAndDistribute` and `burnFromAndDistribute` can not be used for distribution

```solidity
    function mintAndDistribute(
        address account,
        uint256 amount
    ) public onlyOwner {
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        mint(account, amount);
    }
```
distributeToAllHolders() can be called only if `_isPastMinDistributionPeriod() == true`. But if its true, `_beforeMintOrBurn` will revert:
```solidity
    function _beforeMintOrBurn() internal view {
        require(
            !_isPastMinDistributionPeriod(),
            "must distribute before minting or burning"
        );
    }
```

Essentially, `mintAndDistribute` can not be used to `distribute`, and can only be used as `mint()`. For `burnAndDistribute`/`burnFromAndDistribute`, it is exactly the same: they can only be used to burn tokens. 

These three functions should be removed, as they don't add any functionality on top of mint/burn.

# L-03 Due to incorrect `require` statement, `releaseManagedNFT` will not revert if the NFT is not present 

`releaseManagedNFT` is supposed to transfer the specified NFT from the contract to the specified address, and revert if the NFT is not in the list of ManagedNFTs. However, the require statement is incorrect, and the function will still be executed successfully regardless.

## Recommended Mitigation Steps
```diff
+       uint i = 0;
-       for (uint i = 0; i < ManagedNFTs.length; i++) {
+       for (; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
-       require(true, "unable to find released NFT in ManagedNFTs");
+       require(i < ManagedNFTs.length + 1, "unable to find released NFT in ManagedNFTs");
```

# L-04 
`setDistributableERC20s` should be disabled is there's an ongoing distribution.

