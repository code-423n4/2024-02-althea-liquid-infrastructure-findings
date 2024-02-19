## [L-01] Owner centralization risks
There are a bunch of centralization risk issues in the `LiquidInfrastructureERC20` and `LiquidInfrastructureNFT` contract that poses malicious owner risks for users of the protocol. These scenarios are listed below:

1. An owner of the NFT can choose to sell their NFT in Opensea marketplace, thereby locking funds in the contract.

2. The `LiquidInfrastructureERC20` owner can add a massive array of ERC20 tokens as distributable tokens as there's no way to enforce strict adhering to a whitelisted ERC20 token list by the protocol which could DoS the `distribute` function. He can easily do this by creating an array of the same ERC20 token.

The instance of code for 2. is seen below:
```js
  function setDistributableERC20s(
    address[] memory _distributableERC20s // @audit any ERC20 token can be set
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
  }
```

3. The `LiquidInfrastructureERC20` owner can honeypot users by removing those users from the allowlist after minting the Liquid ERC20 token to them initially. 


## [L-02] The function `releaseManagedNFT` should revert if a `managedNft` is not found

The `releaseManagedNFT` function is used to release a NFT which is defined by the parameter `address nftContract` but the function doesn't revert when the NFT is not found inside the array `ManagedNFTs`. The function should revert if the `nftContract` to remove is non-existent. Else, this means that the wrong NFT is being sent as it wasnt added the right way.

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
            if (managed == nftContract) { // @audit loops for nothing if the `nftContract` to remove is not found.
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

## [L-03] Project names being created by `LiquidInfrastructureNFT` can be frontrun

The contructor in the `LiquidInfrastructureNFT` contract constructs the underlying ERC721 with a URI such as `"althea://liquid-infrastructure-account/{accountName}"`, and a symbol `"LIA:{accountName}"`. But the `accountName` can be frontrun by others resulting in a case where the name is taken by another project. This can cause confusion for users as both contract have the `URI`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L62-L74

```solidity
  constructor(
        string memory accountName
    )
    // @audit this can be frontrun
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

## [L-04] Fee-on-transfer tokens can reduce the revenue of holders

Tokens like USDC and USDT utilize upgradeable proxies for their contracts and can become fee-on-transfer tokens in the future if they choose to. On depositing, such fee on transfer tokens can reduce the revenue of holders. The recommendation is to reduce internal transfers as this can eat up into the holder rewards. Rather than sending the revenue from the `LiquidInfrastructureNFT` contract to the `LiquidInfrastructureERC20` allow a direct calculation and transfer from the NFT contract to the holders from the `LiquidInfrastructureERC20` contract.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376-L378

```js
(address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
    withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
    emit Withdrawal(address(withdrawFrom));
```

## [L-05] Expose some getter functions

Add a getter function to retrieve list of all holder address. In the current implementation of the `LiquidInfrastructureERC20` contract, the `holders` array is private. Imagine the owner of the `LiquidInfrastructureERC20` token tells Bob they're an approved holder after a mint of the token but how does Bob verify this information onchain without being tech-savvy to access the private data. There is no way for him to do that. Expose a getter function for the `holders` array.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48

```js
  address[] private holders;
```

## [L-06] Protocol uses very old version of Openzeppelin Library.

Using an old version can lead to worse gas performance, known contract issues, and potential 0-day issues.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/package.json#L23

```js
@>     "@openzeppelin/contracts": "4.3.1",
```

## [L-07] `withdrawFromManagedNFTs` should restrict access to only the owner the `LiquidInfrastructureERC20` contract
With the current implementation of the `withdrawFromManagedNFTs` function, a managed NFT owner can choose to withdraw at any time. This function is better suited to be restricted to the owner of the `LiquidInfrastructureERC20` contract. The owner of `LiquidInfrastructureNFT` can thus easily decide with owner of `LiquidInfrastructureERC20` to withdraw the tokens to himself for a round for example using `LiquidInfrastructureNFT::withdrawBalances`. Currently, anyone can run `LiquidInfrastructureERC20::withdrawFromManagedNFTs` to withdraw the tokens to the `LiquidInfrastructureERC20` contract.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359-L387

```js
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

## [NC-01] Remove unnecessary inherits
The `ERC20` contract does not need to be imported again when it is already imported in the `ERC20Burnable` contract. Similarly, `OwnableApprovableERC721` already imports the `ERC721` implementation. Thereby re-importing is redundant.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L31

```js
  contract LiquidInfrastructureERC20 is
  @>    ERC20,
      ERC20Burnable,
      Ownable,
      ERC721Holder,
      ReentrancyGuard
  {...}
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L33

```js
 @> contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {...}

```

## [NC-02] Remove unnecessary checks from functions

There are many instances in which unnecessary checks are implemented which just increases the gas cost and byte code of the contracts

1. `_isPastMinDistributionPeriod` and `mint` can never be run together. The check highlighted in the code is unnecessary.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303

```js
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

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L344-L349

```js
    function burnFromAndDistribute(address account, uint256 amount) public {
@>        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        burnFrom(account, amount);
    }
```

3. The `require` check in the highlighted code does nothing. It should be removed from the function.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413-L434

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
@>        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }

```

## [NC-03] `thresholdAmounts` are not always equal to the amount received by the `LiquidNFT` contract.
Right now, nothing prevents the contract `LiquidInfrastructureNFT` from receiving more funds than the `thresholdAmounts`. This array is not useful as it is used right now.
