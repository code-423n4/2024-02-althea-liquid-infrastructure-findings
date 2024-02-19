# Gas Optimizations Report

| ID     | Gas Optimization                                                                                           |
|--------|------------------------------------------------------------------------------------------------------------|
| [G-01] | Cache newERC20s.length at the start of function setThresholds()                                            |
| [G-02] | Consider using one setHolder() function instead of approveHolder() and disapproveHolder()                  |
| [G-03] | Instead of using two negative conditions use one positive condition                                        |
| [G-04] | Remove redundant check for LockedDistribution from _beginDistribution() function                           |
| [G-05] | Remove redundant check for erc20EntitlementPerUnit.length > 0 from function _beginDistribution()           |
| [G-06] | Consider using delete instead of assigning false to LockedDistribution                                     |
| [G-07] | Remove redundant check from mintAndDistribute(), burnAndDistribute() and burnFromAndDistribute() functions |
| [G-08] | Non-reentrant modifier can be removed from mint() function                                                 |
| [G-09] | Remove redundant statement from function releaseManagedNFT()                                               |

**Note: None of the issues/instances in this report are included in the bot report.**

## [G-01] Cache newERC20s.length at the start of function setThresholds()

The newERC20s.length storage read is done twice on Line 114 and Line 123. Consider caching the value of this at the start of the function to save gas.
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
123:         for (uint i = 0; i < newErc20s.length; i++) {
124:             thresholdErc20s.push(newErc20s[i]);
125:             thresholdAmounts.push(newAmounts[i]);
126:         }
127:         emit ThresholdsChanged(newErc20s, newAmounts);
128:     }
```

## [G-02] Consider using one setHolder() function instead of approveHolder() and disapproveHolder()

The functions below can be combined into one setHolder() function that takes in `address holder` and `bool approve` as parameters. For approvals, set `approve` to true while for disapprovals set to false.

This would save deployment cost mainly.
```solidity
File: LiquidInfrastructureERC20.sol
108:     function approveHolder(address holder) public onlyOwner {
109:         require(!isApprovedHolder(holder), "holder already approved");
110:         HolderAllowlist[holder] = true;
111:     }
112: 
113:     /**
114:      * Marks `holder` as NOT approved to hold the token, preventing them from receiving any more of the underlying ERC20.
115:      *
116:      * @param holder the account to add to the allowlist
117:      */
118:     function disapproveHolder(address holder) public onlyOwner {
119:         require(isApprovedHolder(holder), "holder not approved");
120:         HolderAllowlist[holder] = false;
121:     }
```

## [G-03] Instead of using two negative conditions use one positive condition

The evaluation of checks can be simplified below.

Instead of this:
```solidity
File: LiquidInfrastructureERC20.sol
144:         bool exists = (this.balanceOf(to) != 0);
145:         if (!exists) {
146:             holders.push(to);
147:         }
```
Use this:
```solidity
File: LiquidInfrastructureERC20.sol
145:         if (this.balanceOf(to) == 0) {
146:             holders.push(to);
147:         }
```

## [G-04] Remove redundant check for LockedDistribution from _beginDistribution() function

The require check below can be removed since the _beginDistribution() function is only called once when LockedForDistribution = false. This is because distribute() function takes care of this [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L200).
```solidity
File: LiquidInfrastructureERC20.sol
259:     function _beginDistribution() internal {
260:         //@audit Gas - redundant check since distribute already takes care of it
261:         require(
262:             !LockedForDistribution,
263:             "cannot begin distribution when already locked"
264:         );
```

## [G-05] Remove redundant check for erc20EntitlementPerUnit.length > 0 from function _beginDistribution()

The check below is not required since at the end of every distribution, the _endDistribution() function deletes the erc20EntitlementPerUnit array [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L291).
```solidity
File: LiquidInfrastructureERC20.sol
268:         if (erc20EntitlementPerUnit.length > 0) {
269:             delete erc20EntitlementPerUnit; 
270:         }
```

## [G-06] Consider using delete instead of assigning false to LockedDistribution

Use delete instead of setting the value to false.
```solidity
File: LiquidInfrastructureERC20.sol
294:         LockedForDistribution = false; 
```

## [G-07] Remove redundant check from mintAndDistribute(), burnAndDistribute() and burnFromAndDistribute() functions

The _isPastMinDistributionPeriod() check is not required in these functions since distribute() function already takes care of it [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L202).
```solidity
File: LiquidInfrastructureERC20.sol
306:     function mintAndDistribute(
307:         address account,
308:         uint256 amount
309:     ) public onlyOwner {
310:         //@audit Gas - check not required since distribute() takes care of it in the start
311:         if (_isPastMinDistributionPeriod()) {
312:             distributeToAllHolders();
313:         }
314:         mint(account, amount);
315:     }

File: LiquidInfrastructureERC20.sol
336:     function burnAndDistribute(uint256 amount) public {
337:         if (_isPastMinDistributionPeriod()) {
338:             distributeToAllHolders();
339:         }
340:         burn(amount);
341:     }
342: 
343:     /**
344:      * Convenience function that allows an approved sender to distribute when necessary and then burn the approved tokens right after
345:      *
346:      * @notice attempts to distribute to every holder in this block, which may exceed the block gas limit
347:      * if this fails then first call distribute() enough times to finish a distribution and then call burnFrom()
348:      */
349:     function burnFromAndDistribute(address account, uint256 amount) public {
350:         if (_isPastMinDistributionPeriod()) {
351:             distributeToAllHolders();
352:         }
353:         burnFrom(account, amount);
354:     }
```

## [G-08] Non-reentrant modifier can be removed from mint() function

The mint() function only makes state changes internally and does not make any external calls that would open up the function to reentrancy. Consider removing the modifier. 
```solidity
File: LiquidInfrastructureERC20.sol
322:     function mint(
323:         address account,
324:         uint256 amount
325:     ) public onlyOwner nonReentrant {
326:         _mint(account, amount);
327:     }
```

## [G-09] Remove redundant statement from function releaseManagedNFT()

The statement below is redundant since it would always evaluate to true due to the hardcoded bool value true. Consider removing the check. 
```solidity
File: LiquidInfrastructureERC20.sol
438:         require(true, "unable to find released NFT in ManagedNFTs");
```