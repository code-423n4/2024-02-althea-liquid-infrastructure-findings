## [L-01] Owner centralization risks.

There are many centralization risk scenario may occur with the owner role of the protocol. The risks are

1. Owner can sell his NFT which will lock the funds.
2. If a scam NFT contract is added to the ManagedNFTs mapping by owner, it could DOS the `distribute` function.

3. Owner can add a bunch of trash ERC20s and DOS `distribute` function.

```solidity
    function setDistributableERC20s(
        address[] memory _distributableERC20s
        ) public onlyOwner {
            distributableERC20s = _distributableERC20s;
    }
```

4. Owner can honeypot users by removing them from allowlist. 
5. Owner can rug-pull the users by adding a rug NFT token.

To avoid such situations, a better decentralized solution is recommended.

## [L-02] The function `releaseManagedNFT` should revert if `managedNft` contract is not available.

The function `releaseManagedNFT` is used to release a NFT which is defined by param `address nftContract` but the function doesn't check the NFT exists or not. The function should revert if the contract is not found.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413

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
        require(true, "unable to find released NFT in ManagedNFTs"); //@audit this require does nothing ? Can an NFT be present here

        emit ReleaseManagedNFT(nftContract, to);
    }

```

## [L-03] The name of account created using the `LiquidInfrastructureNFT` contract can be frontrunned.

The contructor in the contract `LiquidInfrastructureNFT` constructs the underlying ERC721 with a URI like `"althea://liquid-infrastructure-account/{accountName}"`, and a symbol like `"LIA:{accountName}"`. But the `accountName` can be frontrunned by others and that name can't be taken by original caller in future.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L62C1-L74C6

```solidity
  constructor(
        string memory accountName
    )
    // @audit this can be frontrunned
        ERC721(
            string.concat(
                "althea://liquid-infrastructure-account/",
                accountName
            ),
            string.concat("LIA:", accountName)
        )
    {
        _mint(msg.sender, AccountId);
    }
```

## [L-04] Fee-on-transfer tokens can reduce the revenue of holders.

Tokens like USDC and USDT are used in the projects. These tokens are fee-on-transfer type of ERC20 tokens. `withdrawFromManagedNFTs` function is used to deposit all the tokens balances into the custody. On depositing fee on transfer tokens can reduce the revenue of holders. The recommendation is to use less internal transfer with fee on transfer tokens.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376C1-L378C52

```solidity
(address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
```

## [NC-01] Add some getter functions.

Add a getter function to list out the holders addresses unlike in current implementation holders are private variables, holder addresses list can't be accessed by owners directly.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48

```solidity
  address[] private holders;
```

## [NC-02] Project usages very old version of Openzeppelin's Libs.

Using an old version can lead to worse gas performance, known contract issues, and potential 0-day issues.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/package.json#L23

```solidity
@>     "@openzeppelin/contracts": "4.3.1",
```

## [NC-03] `thresholdAmounts` are not always equal to the amount received by the `LiquidNFT` contract.

## [NC-04] `withdrawFromManagedNFTs` should restrict access as owner of NFT can decide to run `withdrawBalances`

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C4-L387C1

```solidity
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

## [NC-05] Remove unnecessary checks from functions.

There are many instances in which unnecessary checks are implemented which just increases the gas cost and byte code of contract.

1. `_isPastMinDistributionPeriod` and `mint` can never be run together. The check highlighted in the code is unnecessary.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303

```solidity
    function mintAndDistribute(
        address account,
        uint256 amount
    ) public onlyOwner {
@>        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        mint(account, amount);
    }
```

2. `_isPastMinDistributionPeriod` and `burnFrom` can never be run together. The check highlighted in the code is unnecessary.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L344C1-L349C6

```solidity
    function burnFromAndDistribute(address account, uint256 amount) public {
@>        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        burnFrom(account, amount);
    }
```

3. The `require` check in the highlighted code does nothing. It should be removed the this function.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413C1-L434C6

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
@>        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }

```

## [NC-06] Remove unnecessary inherits from contracts.

All the functionality of ERC20 are included in the ERC20Burnable of OpenZeppelin with an extra feature that owner can burn all of his tokens. Similarly, OwnableApprovableERC721 contains all the functionality of ERC721 of OpenZeppelin. So it recommended to consider inheriting from `ERC20Burnable` and `OwnableApprovableERC721` only.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L31

```solidity
contract LiquidInfrastructureERC20 is
@>    ERC20,
@>    ERC20Burnable,
    Ownable,
    ERC721Holder,
    ReentrancyGuard
{...}
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L33

```solidity
contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {...}

```

## [NC-08] Typos

The comment should be `ownerOf` instead of `onwerOf` here.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L9

````js
/**
 * An abstract contract which provides onlyOwner(id) and onlyOwnerOrApproved(id) modifiers derived from ERC721's
 @> * onwerOf, getApproved, and isApprovedForAll functions
 * // @audit-info should ownerOf instead of onwerOf[nc]
 * @dev For contracts with manually enumerated tokens (e.g. RockId = 1, PaperId = 2, ScissorsId = 3) use the modifiers like so:
 * ```
 *      // Only the owner of RockId gets to use this function, regardless of approval status
 *      function throwRock() public onlyOwner(RockId) { ... }
 *
 *      // Either the owner, a sender approved for all the tokens the owner of PaperId, or a sender approved for specifically PaperId gets to call this
 *      function wrapWithPaper() public onlyOwnerOrApproved(PaperId) { ... }
 * ```
 */
````
