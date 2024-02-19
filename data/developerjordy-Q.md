### Low Risk Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| [L-01] | No checks for address 0 | 2 |
| [L-02] | Unsafe use of transfer | 1 |
| [L-03] | Wrong implementation of `_isPastMinDistributionPeriod` | 1 |
| [L-04] | Missing address 0 check in constructor loop | 1 |
| [L-05] | require statement always pass | 1 |

## [L-01] No checks for address 0
While deploying the `LiquidInfrastructureERC20` contract there is no check for address 0. similar to the `setDistributableERC20s` method. 

### Recommendations
Add a sanity check for address 0

```diff
constructor:
for (uint i = 0; i < _approvedHolders.length; i++) {
+   require(_approvedHolders[i] != address(0), "address zero");
>    HolderAllowlist[_approvedHolders[i]] = true;
}
```

```diff
function setDistributableERC20s(
    address[] memory _distributableERC20s
) public onlyOwner {
+    address[] memory newList = new address[](_distributableERC20s.length);
+    for (uint i = 0; i < _distributableERC20s.length; i++) {
+       require(_distributableERC20s[i] != address(0), "address zero");
+        newList.push(_distributableERC20s[i])
+    }
+    distributableERC20s = newList; 
-   distributableERC20s = _distributableERC20s;
}
```

## [L-02] Unsafe use of transfer
Not all ERC20 transfers do not return boolean, in this specific scenario it will result to DOS as the transaction will revert. 

Also it is recommeded to use `safeTransfer` instead of the transfer method. 

```solidity
function _withdrawBalancesTo(
    address[] calldata erc20s,
    address destination
) internal {
    uint256[] memory amounts = new uint256[](erc20s.length);
    for (uint i = 0; i < erc20s.length; i++) {
        address erc20 = erc20s[i];
        uint256 balance = IERC20(erc20).balanceOf(address(this));
        if (balance > 0) {
            // @audit-info: not using safeTransfer
>            bool result = IERC20(erc20).transfer(destination, balance); // @audit-info not all tokens returns bool
            require(result, "unsuccessful withdrawal");
            amounts[i] = balance;
        }
    }
    emit SuccessfulWithdrawal(destination, erc20s, amounts);
}
```

### Recommended mitigation steps
Use the SafeERC20 library [implementation]("https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol") from OpenZeppelin and call safeTransfer or safeTransferFrom when transferring ERC20 tokens.


## [L-03] Wrong implementation of `_isPastMinDistributionPeriod`
The comment above the `_isPastMinDistributionPeriod` does not match the implementation of the function. 

The comment tells if the minimun distribution period has elapsed, yet it is compared with the `>=` what results in that the distribution has not officially passed. 

### Recommended mitigation steps
```diff
/**
* Determines if the minimum distribution period has elapsed, which is used for restricting
* minting and burning operations
*/
function _isPastMinDistributionPeriod() internal view returns (bool) {
    // Do not force a distribution with no holders or supply
    if (totalSupply() == 0 || holders.length == 0) {
        return false;
    }

-   return (block.number - LastDistribution) >= MinDistributionPeriod;
+   return (block.number - LastDistribution) > MinDistributionPeriod;
}
```


## [L-04] Missing address 0 check in constructor loop

```diff
constructor(
    string memory _name,
    string memory _symbol,
    address[] memory _managedNFTs,
    address[] memory _approvedHolders,
    uint256 _minDistributionPeriod, // @audit-info: missing NatSpec comment: documented
    address[] memory _distributableErc20s
) ERC20(_name, _symbol) Ownable() {
    ManagedNFTs = _managedNFTs;
    LastDistribution = block.number;

    for (uint i = 0; i < _approvedHolders.length; i++) {
+       require(_approvedHolders[i] != address(0), "Non address zero allowed");
        HolderAllowlist[_approvedHolders[i]] = true;
    }

    MinDistributionPeriod = _minDistributionPeriod; // 1000 blocks

    distributableERC20s = _distributableErc20s; // USDC, DAI,

    emit Deployed();
}
```

## [L-05] require statement always true
The `LiquidInfrastructureERC20::releaseManagedNFT` includes a require statement that will always pass. This is because `true` is hardcoded in the require statement.

Based on the code comments it should only pass when the NFT was found and removed from the `ManagedNFTs` address array. 

### Recommended mitigation steps

```diff
function releaseManagedNFT(
    address nftContract,
    address to
) public onlyOwner nonReentrant {
    LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
    nft.transferFrom(address(this), to, nft.AccountId());
+   bool isFound = false;
    // Remove the released NFT from the collection
    for (uint i = 0; i < ManagedNFTs.length; i++) {
        address managed = ManagedNFTs[i];
        if (managed == nftContract) {
+           isFound = true;
            // Delete by copying in the last element and then pop the end
            ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
            ManagedNFTs.pop();
            break;
        }
    }
    // By this point the NFT should have been found and removed from ManagedNFTs
+    require(isFound, "unable to find released NFT in ManagedNFTs");
-    require(true, "unable to find released NFT in ManagedNFTs");

    emit ReleaseManagedNFT(nftContract, to);
}
```



### Non-Critical Issues List

| Number | Issues Details | Context |
| --- | --- | --- |
| [N-01] | Missing NatSpec comment in constructor | 1 |
| [N-02] | Unnecessary complex logic | 1 |


## [N-01] Missing NatSpec comment in constructor
The constructor lacks missing NatSpec comment for the `uint256 _minDistributionPeriod` parameter. 

```solidity
/**
  * Constructs the underlying ERC20 and initializes critical variables
  *
  * @param _name The name of the underlying ERC20
  * @param _symbol The symbol of the underlying ERC20
  * @param _managedNFTs The addresses of the controlled LiquidInfrastructureNFT contracts
  * @param _approvedHolders The addresses of the initial allowed holders
  * @param _distributableErc20s The addresses of ERC20s which should be distributed from ManagedNFTs to holders
*/ 
constructor(
    string memory _name,
    string memory _symbol,
    address[] memory _managedNFTs,
    address[] memory _approvedHolders,
    uint256 _minDistributionPeriod, // @audit-info: missing NatSpec comment
    address[] memory _distributableErc20s
) ERC20(_name, _symbol) Ownable() {}
```



## [N-02] complex logic should be simplified
In the `LiquidInfrastructureERC20::_beforeTokenTransfer` there's an if statement checking if the `to` address is not zero. 

```solidity
if (!(to == address(0))) {
    require(
        isApprovedHolder(to),
        "receiver not approved to hold the token"
    );
}
```

this could be more simplified by changing the code to
```solidity
if (to != address(0)) {
    require(
        isApprovedHolder(to),
        "receiver not approved to hold the token"
    );
}
```