# QA Report

### Table of Contents

| Number | Issue | Instances |
|--------|-------|-----------|
| [01](#01---call-parent-hook-when-overriding-hooks)     | Call parent hook when overriding hooks      | 2          |
| [02](#02---_aftertokentransfer-does-not-remove-duplicate-elements-properly)     | `_afterTokenTransfer()` does not remove duplicate elements properly      | 1          |
| [03](#03---use-a-pull-over-push-to-avoid-gas-fees-and-potential-dos)     | Use a pull over push to avoid gas fees and potential DoS      | 1          |
| [04](#04---releasemanagednfts-check-for-unable-to-find-nft-does-not-work)     | `releaseManagedNFT()`'s check for 'unable to find NFT' does not work      | 1          |
| [05](#05---ensure-consistent-notices-for-approvedisapprove-functions)     | Ensure consistent notices for approve/disapprove functions      | 1          |
| [06](#06---owner-can-add-duplicate-nfts-to-managednfts)     | Owner can add duplicate NFTs to `ManagedNFTs`      | 1          |
| [07](#07---no-need-to-inherit-from-erc20-when-contract-already-is-erc20burnable)     | No need to inherit from `ERC20` when contract already is `ERC20Burnable`      | 1          |
| [08](#08---consider-changing-scope-for-useful-state-variables-to-public)     | Consider changing scope for useful state variables to `public`      | 2          |
| [09](#09---using-double--reduces-code-readibility)     | Using double `!=` reduces code readibility      | 1          |
| [10](#10---owner-can-add-duplicate-distributableerc20s-with-setdistributableerc20s)     | Owner can add duplicate distributableERC20s with `setDistributableERC20s`      | 1          |

### 01 - Call parent hook when overriding hooks

OpenZeppelin recommends that [hooks should call their parent hook](https://docs.openzeppelin.com/contracts/3.x/extending-contracts#rules_of_hooks) to ensure all hooks in inheritance tree are called.

Affected code (2 instances):
```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
            require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127-L146

```solidity
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
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164-L179

### 02 - `_afterTokenTransfer()` does not remove duplicate elements properly

After a token transfer, `holders` which keeps track of addresses with a non-zero token balance is updated to remove the address `from` if it has a balance of 0. However, if `holders` contain duplicate addresses, it will not remove all addresses due to the way it's implemented. For example, if the addresses are `[a,b,b]`, then upon reaching the first `b` (index 1), it will set index 1 to the last element which is also `b`, then delete the last element. This results in `b` still being present in the array.

Although duplicate addresses should not occur due to the hooks, it is possible to get duplicate addresses by calling `mint()` multiple times to get multiple zero addresses, or by transferring 0 tokens.

Recommended fix is to either ensure no duplicate addresses may occur, or to use a different implementation of removing array elements.

Affected code (1 instance):
```solidity
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
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164-L179

### 03 - Use a pull over push to avoid gas fees and potential DoS

Currently reward tokens are distributed by the contract, which uses an unbounded loop which can fail if the transaction exceeds the block gas limit. The transaction can also fail if a `transfer()` call fails, which will cause a temporary DoS until that token is removed by the owner.

`transfer()` can fail in many ways, including if an approved holder is blacklisted from, for example, USDC, which the sponsor mentioned they will be most likely using.

Additionally, the caller to `distribute()` pays forward all gas required, which reduces their incentive to collect the rewards if they must pay for other's reward token transfer gas fees as well.

To fix this, a [pull over push](https://fravoll.github.io/solidity-patterns/pull_over_push.html) pattern should be used for users to withdraw their tokens, rather than being distributed it.

Affected code (1 instance):
```solidity
    function distribute(uint256 numDistributions) public nonReentrant {
        require(numDistributions > 0, "must process at least 1 distribution");
        if (!LockedForDistribution) {
            require(
                _isPastMinDistributionPeriod(),
                "MinDistributionPeriod not met"
            );
            _beginDistribution();
        }

        uint256 limit = Math.min(
            nextDistributionRecipient + numDistributions,
            holders.length
        );

        uint i;
        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }

                emit Distribution(recipient, distributableERC20s, receipts);
            }
        }
        nextDistributionRecipient = i;

        if (nextDistributionRecipient == holders.length) {
            _endDistribution();
        }
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198-L237

### 04 - `releaseManagedNFT()`'s check for 'unable to find NFT' does not work

There is a `require(true)` statement in `releaseManagedNFT()` which has no effect on the code, and the comment above and error string suggests that it should throw if the NFT specified was not found in `ManagedNFTs`.

Consider removing this, or implementing this with a proper check (suggestion below).

Affected code (1 instance):
```solidity
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
...
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
...
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413-L434

Suggested change:
```diff
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
...
        // Remove the released NFT from the collection
+       bool found;
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
+               found = true;
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
-       require(true, "unable to find released NFT in ManagedNFTs");
+       require(found, "unable to find released NFT in ManagedNFTs");
...
    }
```

### 05 - Ensure consistent notices for approve/disapprove functions

The `approveHolder` function has an `@notice` about how the call can fail. The same notice should be added for `disapproveHolder`, where the call can fail if the holder is not approved.

Affected code (1 instance):
```solidity
    /**
     * Adds `holder` to the list of approved token holders. This is necessary before `holder` may receive any of the underlying ERC20.
     * @notice this call will fail if `holder` is already approved. Call isApprovedHolder() first to avoid mistakes.
     *
     * @param holder the account to add to the allowlist
     */
    function approveHolder(address holder) public onlyOwner {
        require(!isApprovedHolder(holder), "holder already approved");
        HolderAllowlist[holder] = true;
    }

    /**
     * Marks `holder` as NOT approved to hold the token, preventing them from receiving any more of the underlying ERC20.
     *
     * @param holder the account to add to the allowlist
     */
    function disapproveHolder(address holder) public onlyOwner {
        require(isApprovedHolder(holder), "holder not approved");
        HolderAllowlist[holder] = false;
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L100-L119

### 06 - Owner can add duplicate NFTs to `ManagedNFTs`

`ManagedNFTs` should be a unique array of NFTs managed by the LiquidInfrastructureERC20 contract. However, there is no check in place preventing the owner from accidentally adding the same NFT. This results in duplicate events emitted for the same NFT, and also only 1 instance of the NFT removed when `releaseManagedNFT()` is called, which may confuse off-chain programs which track these events.

Consider adding a check to make sure the NFT is not already added.

Affected code (1 instance):
```solidity
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
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394-L403

Suggested change:
```diff
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
+       uint length = ManagedNFTs.length;
+       for (uint i; i<length; i++){
+           if (nftContract == ManagedNFTs[i]){
+               revert("NFT is already in ManagedNFTs");
+           }
+       }
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```

### 07 - No need to inherit from `ERC20` when contract already is `ERC20Burnable`

`ERC20Burnable` already inherits from `ERC20`, thus there is no need for `LiquidInfrastructureERC20` to inherit from `ERC20` again, as it already is `ERC20Burnable`.

Affected code (1 instance):
```solidity
contract LiquidInfrastructureERC20 is
    ERC20,
    ERC20Burnable,
    Ownable,
    ERC721Holder,
    ReentrancyGuard
{
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L29-L35

### 08 - Consider changing scope for useful state variables to `public`

Currently, `holders` and `nextDistributionRecipient` have `private` and `internal` visibility set respectively. This makes it inconvenient for holders trying to withdraw just their reward token using `distribute()`, as they cannot easily see how much they need to `distribute()` until they receive their rewards. Consider making these variables public.

Affected code (2 instances):
```solidity
    address[] private holders;
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48

```solidity
    uint256 internal nextDistributionRecipient;
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L85

### 09 - Using double `!=` reduces code readibility

The below code adds `to` to `holders` if it has a balance of `0`.
```solidity
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
```

This can be a unintuitive as `!=` is used twice - the variable can be removed and replaced with a comment for better readibility, such as:

```diff
-       bool exists = (this.balanceOf(to) != 0);
+       // add `to` to `holders` if it has 0 balance
-       if (!exists) {
+       if (this.balanceOf(to) == 0) {
            holders.push(to);
        }
```

Affected code (1 instance):
```solidity
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L142-L145

### 10 - Owner can add duplicate distributableERC20s with `setDistributableERC20s`

There is no check preventing `distributableERC20s` to contain duplicate addresses when calling `setDistributableERC20s`. If the owner accidentally includes an address twice, this leads to duplicate rewards being distributed which may cause users to lose funds, and `distribute()` to be DoSed as contract lacks funds to fulfill the rewards. Consider adding a check or clear notice for the array to consist of unique elements

Affected code (1 instance):
```solidity
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445