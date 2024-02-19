# Quality Assurance Report

| ID     | Gas Optimization                                                                |
|--------|---------------------------------------------------------------------------------|
| [L-01] | Function setThresholds() might DOS due to OOG error                             |
| [L-02] | Missing address(0) check when approving holder through approveHolder() function |
| [L-03] | Consider restricting transfers once minDistributionPeriod has passed            |
| [L-04] | Distribution would DOS if erc20EntitlementPerUnit array grows too large         |
| [L-05] | Convenience functions can be frontrun with distribute() function calls          |
| [L-06] | Function addManagedNFT() does not check if nftContract already exists           |
| [L-07] | Function releaseManagedNFT() could permanently DOS due to out-of-gas error      |
| [N-01] | Consider using block.timestamp instead of block.number                          |

## [L-01] Function setThresholds() might DOS due to OOG error

New thresholds cannot be set if the previous thresholds set through this function make the array thresholdErc20s or thresholdAmounts too long to delete. This will cause a DOS when the delete operation is being performed due to out-of-gas error.
```solidity
File: LiquidInfrastructureNFT.sol
109:     function setThresholds(
110:         address[] calldata newErc20s,
111:         uint256[] calldata newAmounts
112:     ) public virtual onlyOwnerOrApproved(AccountId) {
113:         require(
114:             newErc20s.length == newAmounts.length,
115:             "threshold values must have the same length"
116:         );
117:         // Clear the thresholds before overwriting
119:         delete thresholdErc20s;
120:         delete thresholdAmounts;
121: 
122:         for (uint i = 0; i < newErc20s.length; i++) {
123:             thresholdErc20s.push(newErc20s[i]);
124:             thresholdAmounts.push(newAmounts[i]);
125:         }
126:         emit ThresholdsChanged(newErc20s, newAmounts);
127:     }
```

## [L-02] Missing address(0) check when approving holder through approveHolder() function

If the owner forgets to pass in the holder address, the value is defaulted to address(0).
```solidity
File: LiquidInfrastructureERC20.sol
107:     function approveHolder(address holder) public onlyOwner {
108:         require(!isApprovedHolder(holder), "holder already approved");
109:         HolderAllowlist[holder] = true;
110:     }
```

## [L-03] Consider restricting transfers once minDistributionPeriod has passed 

The below check ensures that once the minimum distribution period passes, mints and burns are prevented ensuring that supply changes happen after any potential distributions. Consider also restricting transfers by adding this function call [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L133).

This would ensure no major/minor transfers can occur before distributions occur. The liquid infrastructure would be used in several way externally and each design/model would require it's own restrictions. 

For example consider a case where the approvedHolder is an address that provides some kind of service to its users. The users (non-technical) are not aware of the actual revenue they should be receiving and are instead fooled by the approvedHolder who informs them of lesser revenue. In this case, the approved holder transfers the tokens to another address right before distribution occurs (but after pastMintDistribution time). He receives the revenue on the other approved address. When the distribution ends, he transfers only a % of tokens to the users, who believe him since the tokens were to be considered non-transferrable once a distribution period begins.

```solidity
File: LiquidInfrastructureERC20.sol
151:     function _beforeMintOrBurn() internal view {
152:         require(
153:             !_isPastMinDistributionPeriod(),
154:             "must distribute before minting or burning"
155:         );
156:     }
```

## [L-04] Distribution would DOS if erc20EntitlementPerUnit array grows too large

Consider limiting the amount of distributableERC20s to ensure erc20EntitlementPerUnit does not grow too large to delete. This would cause the distribution to DOS (even though batch distributions are implemented) and cause approved holders revenue to be permanently stuck in the contract.

The erc20EntitlementPerUnit grows large based on distributableERC20s as seen [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L271).
```solidity
File: LiquidInfrastructureERC20.sol
286:     function _endDistribution() internal {
287:         require(
288:             LockedForDistribution,
289:             "cannot end distribution when not locked"
290:         );
291:         delete erc20EntitlementPerUnit; 
292:         LockedForDistribution = false; 
293:         LastDistribution = block.number;
294:         emit DistributionFinished();
295:     }
```

## [L-05] Convenience functions can be frontrun with distribute() function calls

If the owner or approved holders call the convenience functions below to start distributions and mint/burn tokens, someone can frontrun their call with a distribute() function call with numDistributions set to a value less than holders.length. This would make them devoid of immediately minting or burning tokens after distributions.

This could be an issue where the approvedHolder provides a service on Althea L1 that requires this kind of functionality to work. 
```solidity
File: LiquidInfrastructureERC20.sol
304:     function mintAndDistribute(
305:         address account,
306:         uint256 amount
307:     ) public onlyOwner {
308:         if (_isPastMinDistributionPeriod()) {
309:             distributeToAllHolders();
310:         }
311:         mint(account, amount);
312:     }

File: LiquidInfrastructureERC20.sol
332:     function burnAndDistribute(uint256 amount) public {
333:         if (_isPastMinDistributionPeriod()) {
334:             distributeToAllHolders();
335:         }
336:         burn(amount);
337:     }
338: 
339:     /**
340:      * Convenience function that allows an approved sender to distribute when necessary and then burn the approved tokens right after
341:      *
342:      * @notice attempts to distribute to every holder in this block, which may exceed the block gas limit
343:      * if this fails then first call distribute() enough times to finish a distribution and then call burnFrom()
344:      */
345:     function burnFromAndDistribute(address account, uint256 amount) public {
346:         if (_isPastMinDistributionPeriod()) {
347:             distributeToAllHolders();
348:         }
349:         burnFrom(account, amount);
350:     }
```

## [L-06] Function addManagedNFT() does not check if nftContract already exists

Consider implementing a mapping from nftContract to bool status that tracks if an nftContract has already been added. Then add a check in the function addManagedNFT() and state updates to true/false in addManagedNFT() and releaseManagedNFT().

This would avoid duplication of an nftContract that has already been added.
```solidity
File: LiquidInfrastructureERC20.sol
396:     function addManagedNFT(address nftContract) public onlyOwner {
397:         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
398:         address nftOwner = nft.ownerOf(nft.AccountId());
399:         require(
400:             nftOwner == address(this),
401:             "this contract does not own the new ManagedNFT"
402:         );
403:         ManagedNFTs.push(nftContract);
404:         emit AddManagedNFT(nftContract);
405:     }
```

## [L-07] Function releaseManagedNFT() could permanently DOS due to out-of-gas error

If the ManagedNFTs array grows too large, the for loop below would DOS due to OOG exception, causing the removal of managed nfts not possible. This is because we iterate upto the length of the array if the nftContract is towards the end of the array.
```solidity
File: LiquidInfrastructureERC20.sol
414:     function releaseManagedNFT(
415:         address nftContract,
416:         address to
417:     ) public onlyOwner nonReentrant {
418:         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
419:         nft.transferFrom(address(this), to, nft.AccountId());
420: 
421:         // Remove the released NFT from the collection
422:         for (uint i = 0; i < ManagedNFTs.length; i++) {
423:             address managed = ManagedNFTs[i];
424:             if (managed == nftContract) {
425:                 // Delete by copying in the last element and then pop the end
426:                 ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
428:                 ManagedNFTs.pop();
429:                 break;
430:             }
431:         }
432:         // By this point the NFT should have been found and removed from ManagedNFTs
433:         require(true, "unable to find released NFT in ManagedNFTs");
434: 
435:         emit ReleaseManagedNFT(nftContract, to);
436:     }
```

## [N-01] Consider using block.timestamp instead of block.number

Although distributions would be weekly/monthly, it is generally recommended to use block.timestamp to ensure that shorter distribution periods (if set in the future or by other team using these contracts) are considered.
```solidity
File: LiquidInfrastructureERC20.sol
465:         LastDistribution = block.number; 
```