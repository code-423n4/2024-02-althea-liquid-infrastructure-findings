# L - 01
Zero-transfer adds the recipient to the holders list

If the account who transfers the token has no balance, they are removed from the holders list. However, there's a transfer of 0 tokens, the recipient will be added to the list of holders.
 
# L - 02 
mintAndDistribute/burnAndDistribute/burnFromAndDistribute will revert if they enter the `if` clause:

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
distributeToAllHolders() can be called only if `_isPastMinDistributionPeriod() == true`. But if its true, _beforeMintOrBurn will revert:
```solidity
    function _beforeMintOrBurn() internal view {
        require(
            !_isPastMinDistributionPeriod(),
            "must distribute before minting or burning"
        );
    }
```
Essentially, mintAndDistribute can not be used to distribute, and can only be used as `mint()`. For burnAndDistribute/burnFromAndDistribute, it is exactly the same.

# L - 03 
wrong require statement

# L - 04 
setDistributableERC20s should be disabled is there's an ongoing distribution

# L - 05
