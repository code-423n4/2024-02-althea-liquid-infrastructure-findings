`Low Risk & Non Critical Findings`
# Report
## Low Risk Findings 
**Low-Risk Findings List**
| Number | Issue Details | Instances |
|-----:|----|-----|
|L-01 | NFT tokens could be lost forever on the call to `releaseManagedNFT()`. | 1 |
|L-02 | Create a function to add to `distributableERC20s` so the owner doesn't always have to overwrite the list of `ERC20`s whenever he/she intends to add more `distributableERC20s` tokens to the contracts. | 1 |
| N-01 | The `constructor()` doesn't check if the contract owns the managed NFTs. | 1 |
| N-02 | Test suite coverage doesn't cover some functions. | 3 |
|N-03 | Redundant function. | 1 |


# L-01 NFT tokens could be lost forever on the call to `releaseManagedNFT()`.
## Summary 
In `LiquidInfrasture::releaseManagedNFT()`, the function use `transferFrom()` to send the NFT to the reciepient:
```solidity
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        //@audit uses transferFrom, nft can be lost forever.
        nft.transferFrom(address(this), to, nft.AccountId());
```
Use Always `safeTransferFrom()`, because in a situation where the to address is a contract and it doesn't implement the `ERC721` Interface there would be no way for the receiver to access the NFT token, causing it to be lost forever.

> [!Warning]
> In a scenario, where the LiquidInfrastructureNFT is lost all the accrued revenue will also be lost.
## Recommended Mitigation.
Make use of OZs `safeTransferFrom()` to transfer the NFTs to the recipientt.
**Sample of fix:**
```diff
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
-        nft.safeTransferFrom(address(this), to, nft.AccountId());
+        nft.safeTransferFrom(address(this), to, nft.AccountId());
```



# L-02 Create a function to add to `distributableERC20s` so the owner doesn't always have to overwrite the list of `ERC20`s whenever he/she intends to add more `distributableERC20s` tokens to the contracts.
## Summary 
In `LiquidInfrastructureERC20`, the owner has to always reset the `distributableERC20s` in any scenario he/she wants to and more distributable tokens to the contracts:
```solidity
  function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```
This can lead to confusions, account issues, and gas spent on the call to reset the `distributableERC20s`.
## Recommended Mitigation.
Create a function to add to the already existent `distributableERC20s` of the contract.
**Sample of fix:**
```solidity
  function addDistributableERC20s(
        address memory _distributableERC20
    ) public onlyOwner {
        require(_distributableERC20 != address(0), "not a token");
        distributableERC20s.push(_distributableERC20);
    }
```


# N-01 The `constructor()` doesn't check if the contract owns the managed NFTs.
## Summary 
During deployment, the `LiquidInfrastructureERC20` cannot check if it is approved of or owns the ManagedNFTs, Inputted in the constructor and it is paramount that the LiquidInfrastructure owns or is approved of those NFT addresses:
```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
@>        ManagedNFTs = _managedNFTs;
```
This can cause issues when calling `withdrawFromManagedNFTs()` if the address isn't checked correctly and it can brick the withdrawal queue.
> [!Note]
> `withdrawFromManagedNFTs()` is the main source of revenue for the `LiquidInFrastructureERC20`, if the withdrawal process is bricked the LiquidInfra Tokens.
## Recommended Mitigation.
don't set the _managedNFTS in the `constructor()`, use `addManagedNFT()` to add the managed NFTs as it implements the appropriate checks.

# N-02 Test suite coverage doesn't cover some edge cases.
## Summary
The tests are well written, however, certain scenarios were not tested like:
- Tokens with weird decimals, as the protocols will interact with lots of different tokens
- Gas testing as functions will have to be called multiple times to execute certain events like
* Distribution
* Withdrawals
Given the amount of loops used in the contract, that could also be recommended.
## Recommended Mitigation.
Perform tests on such cases that are likely to occur in the protocol.

# N-02 Redundant function.
## Summary 
The `mint()` and `mintAndDistribute()` contain the same logic, except that the `mintAndDistribute()` checks if `_isPastMinDistributionPeriod()` and attempts to distribute if `true`:
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


  function mint(
        address account,
        uint256 amount
    ) public onlyOwner nonReentrant {
        _mint(account, amount);
    }

```

The thing here is that, `mint()` will always revert if the `_isPastMinDistributionPeriod()` returns `true`, so the caller will still have to manually call `distribute()` before the can successfully call `mint()`. this is because of the hooks placed when minting and burning.
```solidity
   function _beforeMintOrBurn() internal view {
        if 
        require(
            !_isPastMinDistributionPeriod(),
            "must distribute before minting or burning"
        );
    }
```
so it is useless to have to minting functions when the both behave extactly the same.
## Recommended Mitigation.
The issue can be solved by clearing the `mint()` function and sticking with the `mintAndDistribute()` 
**sample of fix**
```diff
Files: LiquidInfrastrucureERC20.sol

    function mintAndDistribute(
        address account,
        uint256 amount
-    ) public onlyOwner {
+    ) public onlyOwner nonReentrant {
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        _mint(account, amount);
    }

-   function mint(
-        address account,
-        uint256 amount
-    ) public onlyOwner nonReentrant {
-       _mint(account, amount);
-   }
```