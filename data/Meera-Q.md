**Table of Contents**

- [1. Low Severity](#1-low-severity)
  - [1.1. Managed NFT can still be held in the contract after release](#11-managed-nft-can-still-be-held-in-the-contract-after-release)
  - [1.2. require statement in `LiquidInfrastructureERC20.releaseManagedNFT()` could be false](#12-require-statement-in-liquidinfrastructureerc20releasemanagednft-could-be-false)
  - [1.3. Don't use `_msgSender()` if not supporting EIP-2771](#13-dont-use-_msgsender-if-not-supporting-eip-2771)
  - [1.4. Using `this.` will change the context to an external call](#14-using-this-will-change-the-context-to-an-external-call)
- [2. Informational](#2-informational)
  - [2.1. `withdrawBalances` and `withdrawBalancesTo`: consider using `onlyOwnerOrApproved`](#21-withdrawbalances-and-withdrawbalancesto-consider-using-onlyownerorapproved)
  - [2.2. Redundant ERC721 inheritance for LiquidInfrastructureNFT](#22-redundant-erc721-inheritance-for-liquidinfrastructurenft)
  - [2.3. Redundant ERC20 inheritance for LiquidInfrastructureERC20](#23-redundant-erc20-inheritance-for-liquidinfrastructureerc20)
  - [2.4. Non-Standard swap \& pop](#24-non-standard-swap--pop)

# 1. Low Severity

## 1.1. Managed NFT can still be held in the contract after release

`LiquidInfrastructureERC20.releaseManagedNFT()` is described as such: `Transfers a LiquidInfrastructureNFT contract out of the control of this contract`. Notice the mention of `out`.

However, if `address to` is equal to the `LiquidInfrastructureERC20` contract itself, the `ManagedNFTs` array won't contain the managed NFT anymore but the contract will still have it.

It's possible to add it again through a call to `addManagedNFT` and release it for real with `to != LiquidInfrastructureERC20`.

The whole thing should be impossible however, so consider adding a check like `to != address(this)` in `LiquidInfrastructureERC20.releaseManagedNFT()`.

## 1.2. require statement in `LiquidInfrastructureERC20.releaseManagedNFT()` could be false

As per the above finding: if the NFT is released to the `LiquidInfrastructureERC20` contract itself, a second call won't find the `released NFT in ManagedNFTs` array, hence proving the `require(true)` to be false, while having a hardcoded `true` value.
Consider using a real boolean in the function to check for the ManagedNFT being removed

```solidity
File: LiquidInfrastructureERC20.sol
431:         require(true, "unable to find released NFT in ManagedNFTs");
```

## 1.3. Don't use `_msgSender()` if not supporting EIP-2771

Use `msg.sender` if the code does not implement [EIP-2771 trusted forwarder](https://eips.ethereum.org/EIPS/eip-2771) support: <https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L26>

## 1.4. Using `this.` will change the context to an external call

Inside `ownerOf`, `msg.sender` will be `OwnableApprovableERC721` instead of the caller:

```diff
File: OwnableApprovableERC721.sol
24:     modifier onlyOwner(uint256 tokenId) {
25:         require(
- 26:             ERC721(this).ownerOf(tokenId) == _msgSender(), 
+ 26:             ownerOf(tokenId) == msg.sender, 
27:             "OwnableApprovable: caller is not the owner"
28:         );
29:         _;
30:     }
```

Additionally, this costs a lot of gas

# 2. Informational

## 2.1. `withdrawBalances` and `withdrawBalancesTo`: consider using `onlyOwnerOrApproved`

Using the `onlyOwnerOrApproved` would be better than duplicating the require statement [LiquidInfrastructureNFT.sol#L136-L139](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L136-L139)

## 2.2. Redundant ERC721 inheritance for LiquidInfrastructureNFT

Because we already have this:

```solidity
File: OwnableApprovableERC721.sol
20: abstract contract OwnableApprovableERC721 is Context, ERC721 {
```

This can be simplified:

```diff
- 33: contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
+ 33: contract LiquidInfrastructureNFT is OwnableApprovableERC721 {
```

## 2.3. Redundant ERC20 inheritance for LiquidInfrastructureERC20

Because we already have this:

```solidity
File: ERC20Burnable.sol
13: abstract contract ERC20Burnable is Context, ERC20 {
```

This can be simplified:

```diff
File: LiquidInfrastructureERC20.sol
29: contract LiquidInfrastructureERC20 is
- 30:     ERC20,
31:     ERC20Burnable,
```

## 2.4. Non-Standard swap & pop

The standard "swap & pop" doesn't swap if `i == length - 1`.

```solidity
    if (i < array.length - 1) {
        array[i] = array[lastIndex];
    }
    array.pop();
```

Consider adding the check on [LiquidInfrastructureERC20.sol#L425](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L425) and [LiquidInfrastructureERC20.sol#L174](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L174)
