# 1. Check for ERC20 balances before releasing the managed NFT

Lines of code* 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413C2-L434C6

## Impact
The `releaseManagedNFT` function transfers the nft to a selected address. It however doesn't check or withdraw the ERC20 token balances in the nft before transferring. Doing this can lead to loss of these tokens if they had not been withdrawn from the nft before transfer. 

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

## Recommended Mitigation Steps

Consider adding a check for the `withdrawERC20s` balance and a call to the `withdrawFromManagedNFTs` function before transferring the nft.

***

# 2. Add a check for 0 `numWithdrawals` when withdrawing from NFTs

Lines of code*
 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C14-L359C37

## Impact
The `withdrawFromManagedNFTs` function withdraws token from the managed NFTs to the liquidInfrastructureErc20 token contract. It however lacks a check for 0 `numWithdrawals` which can lead to the function running without actually doing anything. 

```

    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;

        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```
## Recommended Mitigation Steps

Consider adding a 0 check and reverting if there are no nft to withdraw from.
***

# 3. Redundant deletion of `erc20EntitlementPerUnit`

Lines of code* 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L266

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L291

## Impact

Upon distribution start and end, the `erc20EntitlementPerUnit` is deleted, making one of the checks redudndant. Consider deleting one of them, preferable the one in the `_endDistribution` function.
```
    function _beginDistribution() internal {
        require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );
        LockedForDistribution = true;

        // clear the previous entitlements, if any
        if (erc20EntitlementPerUnit.length > 0) {
            delete erc20EntitlementPerUnit;
        }

```
```
    function _endDistribution() internal {
        require(
            LockedForDistribution,
            "cannot end distribution when not locked"
        );
        delete erc20EntitlementPerUnit;
        LockedForDistribution = false;
        LastDistribution = block.number;
        emit DistributionFinished();
    }
```

***

# 4. Redundant check in the `distributeToAllHolders` function should be removed.

Lines of code* 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184C1-L189C6

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L199

## Impact

The `distributeToAllHolders` function makes a check to see if number of holders is more than zero, upon which it calls the `distribute` function.

```
    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
            distribute(holders.length);
        }
    }
```
The check is not needed as the `distribute` function also checks for non zero `numnumDistributions`. 
```
  function distribute(uint256 numDistributions) public nonReentrant {
        require(numDistributions > 0, "must process at least 1 distribution");
        ...
```
Consider removing the check and passing the values in directly.

***

# 5. `addManagedNFT` and constructor should include a check for duplicate NFTs

Lines of code* 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394C1-L403C6

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413C1-L434C6

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C1-L386C6

## Impact
Minor impact is that gas will be burned during zero amount transfers. 
Major impact is a DOS upon calling the `withdrawFromManagedNFTs` function after the NFT is released.

The `addManagedNFT` function allows the owner to add the LiquidInfrastructureNFTs to the list of managed NFTs, upon which the NFT is added to the list of the protocols's NFTs. The function and the constructor lacks sanity checks, which can cause that nfts can be duplicated.

```
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```

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
        LastDistribution = block.number;

        for (uint i = 0; i < _approvedHolders.length; i++) {
            HolderAllowlist[_approvedHolders[i]] = true;
        }

        MinDistributionPeriod = _minDistributionPeriod;

        distributableERC20s = _distributableErc20s;

        emit Deployed();

    }
```

For the minor impact, upon calling the `withdrawFromAllManagedNFTs`/`withdrawFromManagedNFTs` function, the `withdrawBalancesTo`  function is called in the `LiquidInfrastructureNFT` contract.

```
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        ...
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
```
```
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```
After the withdrawal from the first duplicate, the ERC20 balance is 0, so the withdrawals from the second/subsequent duplicates are skipped.

For the more major impact, first the `releaseManagedNFT` is called. This first transfers the NFT, and then deletes the NFT from the array of managedNFTs. However, due to the use of the `break` functionality, it doesn't a loop through the entire array of NFTs, but rather stops at the first discovered NFT, i.e , the first duplicate. Leaving the remaining duplicates in the managedNFTs array. 

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
Now, important to note that, the NFT transfer transfers the ownership to the other address, causing that the LiquidInfrastructureERC20 contract is no longer the token owner.

Upon calling `withdrawFromAllManagedNFTs`/`withdrawFromManagedNFTs` function, the `withdrawBalancesTo`  function is called in the `LiquidInfrastructureNFT` contract. The `withdrawBalancesTo` function can only be called by the owner or an approved user, hence the `LiquidInfrastructureERC20` contract will not be able to call this contract, thus dossing the nft balance withdrawal process. 

```
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

This duplicate token can also not be removed by calling the `releaseManagedNFT` function, as the `transferFrom` function has to be called by either the owner or an approved user. 

Ultimately, this depends on the owner mistake, but can have serious effects in the protcol, so I'd leave this up to the judge.
## Recommended Mitigation Steps
Consider adding checks for duplications, or removing the `break` functionality from the `releaseManagedNFT` function, to allow for full looping and full removal of all duplicates.

***

# 6. Ineffeective check in the `releaseNFT` function

Lines of code* 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431

## Impact

This check is redundant and does not serve any purpose as it always evaluates to true. It should ideally check if the NFT was successfully removed from ManagedNFTs, but it is incorrectly written.

```
   function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
...
        require(true, "unable to find released NFT in ManagedNFTs");
        emit ReleaseManagedNFT(nftContract, to);
    }

```
## Recommended Mitigation Steps
Consider refactoring to something like this

```

    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());
        bool foundAndRemoved = false;  
        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                foundAndRemoved = true;
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```
***

# 7. Tokens can be lost in the contract if the `withdrawERC20s` are not set as `distributableERC20s` 

Lines of code* 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L473

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441C1-L445C6

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108C1-L125C6

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L377

## Impact

The constructor and the `setDistributableERC20s` allows the owner of the LiquidInfrastructureERC20 contract to set the ERC20 reward tokens to distribute. 

```
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```
```
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
```
The owner of the LiquidInfrastructureNFT contract also has the ability to set the ERC20s as thresholds 
```
    function setThresholds(
        address[] calldata newErc20s,
        uint256[] calldata newAmounts
    ) public virtual onlyOwnerOrApproved(AccountId) {
        require(
            newErc20s.length == newAmounts.length,
            "threshold values must have the same length"
        );
        // Clear the thresholds before overwriting
        delete thresholdErc20s;
        delete thresholdAmounts;

        for (uint i = 0; i < newErc20s.length; i++) {
            thresholdErc20s.push(newErc20s[i]);
            thresholdAmounts.push(newAmounts[i]);
        }
        emit ThresholdsChanged(newErc20s, newAmounts);
    }

```
which can later be withdrawn into the LiquidInfrastructureERC20 contract.

```
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;

        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```
As there's no specific check, or enforcements that the `withdrawERC20s` are also set as parts of the `distributableERC20s`, these tokens being withdrawn into the LiquidInfrastructureERC20 stand the risk of being stranded if they're not made distributable, as there's no other way to retrieve tokens from the contract.

## Recommended Mitigation Steps
Consider introducing a sweep function in the LiquidInfrastructureERC20 contract, or adding an using the `getThresholds` function to get the array of `withdrawERC20s` and enforcing them as part of the `distributableERC20s`
***

# 8. Contracts cannot be upgraded

Lines of code* 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L50C1-L54C41

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L4C1-L12C1
## Impact

The `LiquidInfrastructureERC20` has a version and judging by the comment, it seems the contracts are intended to be upgradeable, so as to seamlessly make updates to the contract while introducing a new version. 
```
    /**
     * @notice This is the current version of the contract. Every update to the contract will introduce a new
     * version, regardless of anticipated compatibility.
     */
    uint256 public constant Version = 1;
```
But the contract doesn't inherit any upgradeable OZ contracts, nor storage gaps.

```
import "@openzeppelin/contracts/utils/math/Math.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "./LiquidInfrastructureNFT.sol";

```
## Recommended Mitigation Steps
If upgradeability is truly desired, inherit the upgradeable OZ contracts, and add storage gaps.

***