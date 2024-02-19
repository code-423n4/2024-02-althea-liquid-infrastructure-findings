
# QA Report

## Summary

| id | title |
| --- | --- |
| [L-1](#l-1-a-new-holder-can-claim-tokens-within-a-same-block-of-minting) | a new holder can claim tokens within a same block of minting|
| [L-2](#l-2-WithdrawalStarted-can-be-emitted-infinite-times-by-anyone) | `WithdrawalStarted` can be emitted infinite times by anyone|
| [L-3](#l-3-tokens-are-withdrawed-from-nfts-even-if-there-are-no-holders) | tokens are withdrawed from nfts even if there are no holders|
| [L-4](#l-4-distribution-action-will-revert-if-bool-is-not-returned-on-token-transfer) | distribution action will revert if bool is not returned on token transfer|
| [L-5](#l-5-_afterTokenTransfer-doen't-break-the-loop-when-the-holder-is-matched) | `_afterTokenTransfer` doen't break the loop when the holder is matched |
| [L-6](#l-6-`nextDistributionRecipient`-is-not-set-to-zero-after-the-end-of-distribution) | `nextDistributionRecipient` is not set to zero after the end of distribution|
| [L-7](#l-7-make-MinDistributionPeriod-time-dependent-rather-than-block-dependent) | make  `MinDistributionPeriod` time dependent rather than block dependent |
| [L-8](#l-8-withdrawFromManagedNFTs-doesn't-validate-if-the-threshold-tokens-are-also-one-of-the-distributableERC20s-tokens) | `withdrawFromManagedNFTs` doesn't validate if the threshold tokens are also one of the `distributableERC20s` tokens|
| [L-9](#l-9-addManagedNFT-doesn't-validate-duplication) | `addManagedNFT` doesn't validate duplication|
| [L-10](#l-10-using-old-deprecated-and-vulnerable-open-zeppelin-lirary) | using old deprecated and vulnerable open-zeppelin lirary|

## Low


### L-1 a new holder can claim tokens within a same block of minting

- When the owner  mints tokens for a holder by calling [mintAndDistribute](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303), that holder can backrun that action by calling [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359)  and then calling distribute again to claim the tokens. So its an unfair reward distribution, by claiming within the same block.
So, it is recommended to not call [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359) whenever a mint/burn happens by validating with `require(!_isPastMinDistributionPeriod)`

From [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359)

```diff
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution"); 
+       require(!_isPastMinDistributionPeriod, "not immediately after distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        
        ---------------------- skimmed ------------------------
    }
```

### L-2 `WithdrawalStarted` can be emitted infinite times by anyone

Any one can call [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359) with 0 as paramets input, and it will emit the event. So, any external offchain event dependent backend can be tricked to do actions whenever an attacker wants it.

From [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359)

```diff
function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution"); 

-       if (nextWithdrawal == 0) {
+       if (nextWithdrawal == 0 && numWithdrawals > 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;

        ---------------------- skimmed ------------------------

    }
```

### L-3 tokens are withdrawed from nfts even if there are no holders

tokens are withdrawn from nfts into the contract to distribute to holders, but they are pulled even if there are no holders or total token supply is 0. So the new minted holder can claim the tokesn that they doon't deserve. So withdraw only when holders array length > 0.

From [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359)

```diff
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
+       require(holders.length > 0);
        require(!LockedForDistribution, "cannot withdraw during distribution"); 

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        
        ---------------------- skimmed ------------------------
    }
```

### L-4 distribution action will revert if bool is not returned on token transfer

The token trasnfer implements below way to validate if token transfer is successful or not. But some tokens does not return bool, making the distribution transaction revert. Instead use safeTransfer which validates if return data well.

From [_withdrawBalancesTo](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L170-L185)
```diff
+   using SafeErc for IERC20;
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
-               bool result = IERC20(erc20).transfer(destination, balance);
-               require(result, "unsuccessful withdrawal");
+               IERC20(erc20).safeTransfer(destination, balance);
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```


### L-5 `_afterTokenTransfer` doen't break the loop when the holder is matched

This will make the transfers of token possible, or else if holders array is huge, then transfers may fail due to out of gas panic reverts.

From [_afterTokenTransfer](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L164)

```diff
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
+                   break();
                }
            }
        }
    }
```



### L-6 `nextDistributionRecipient` is not set to zero after the end of distribution

`nextDistributionRecipient` is set in order to start the next distribution cycle., but it is not set to zero afte the end of distribution cycle. So any external reading contracts will rely on wrong data. Plus settting a state to zero will refund some gas on ethereum.

From [distribute](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L234)

```diff
function distribute(uint256 numDistributions) public nonReentrant {
        require(numDistributions > 0, "must process at least 1 distribution");
        
        ----------------- skimmed ----------------------

        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) { 
                uint256[] memory receipts = new uint256[](distributableERC20s.length);
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
+           nextDistributionRecipient = 0;
        }
    }
```

### L-7 make `MinDistributionPeriod` time dependent rather than block dependent

- Contract counts minimuim number blocks it needs to pass in order to start distribution again. 
- Instead of counting the blocks which can change its block time according to the governance, we should rely on block timestamp.
- so modify the usage of `uint256 public MinDistributionPeriod`  into block.timestamp.



### L-8 `withdrawFromManagedNFTs` doesn't validate if the threshold tokens are also one of the `distributableERC20s` tokens

- If any token that is not in `distributableERC20s` returned by `withdrawFrom.getThresholds` will pull that token, and it wont be distrubuted, so it will be locked there until that token is included in the `distributableERC20s`, so holders who did not hold when pulled will also share those rewards. Which is unfair.

Modify [withdrawFromManagedNFTs](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359) to pull tokens that are in the `distributableERC20s` array.



### L-9 `addManagedNFT` doesn't validate duplication

`addManagedNFT` does not check if a manager is already added by the owner. THis duplicate will make temporary dos of reverting in withdraw action. Will become solved, if owner releases the dupe.

from [addManagedNFT](https://github.com/code-423n4/2024-5-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394)

```diff
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );

+       for (uint i = 0; i < ManagedNFTs.length; i++) {
+           address managed = ManagedNFTs[i];
+           require(managed != nftContract) 
+       }

        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```



### L-10 using old deprecated and vulnerable open-zeppelin lirary
- The contracts are using 4.3.1 version that has vulnerabilities across some contracts. So its betetr to use the latest version 5.0.0.
Check the vulnerablilities in 4.3.1 [here](https://github.com/GeorgeHNTR/oz-known-issues?tab=readme-ov-file#431-14)