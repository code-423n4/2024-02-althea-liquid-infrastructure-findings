# Codebase Optimization Report

## Table of Contents

- [Codebase Optimization Report](#codebase-optimization-report)
  - [Table of Contents](#table-of-contents)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Note on Gas estimates.](#note-on-gas-estimates)
  - [Redundant checks and calls due to the variable passed as argument(Save 3324 Gas on average)](#redundant-checks-and-calls-due-to-the-variable-passed-as-argumentsave-3324-gas-on-average)
  - [Unnecessary checks being performed due to the arguement passed(Save 621 Gas on average)](#unnecessary-checks-being-performed-due-to-the-arguement-passedsave-621-gas-on-average)
  - [Pack `lastDistribution` with `lockedForDistribution`(Save 1 SLOT: 2000 Gas)](#pack-lastdistribution-with-lockedfordistributionsave-1-slot-2000-gas)
  - [Conclusion](#conclusion)


## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.


## Note on Gas estimates.
I've tried to give the exact amount of gas being saved from running the included tests. Whenever the function is within the test coverage, the average gas before and after will be included, and often a diff of the code will also accompany this.

The command to get the gas used `REPORT_GAS=true npm run test`


**This report only focuses on higher impact savings ie quality over quantity**


## Redundant checks and calls due to the variable passed as argument(Save 3324 Gas on average)
**Gas before and after**
|        | Min  | Max | AVG | 
| ------ | ---- | ------- | ------ |
| Before | 23422 | 2273532 | 603821
| After  | 23422 | 2270413  |600497        |
  
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184-L189
```solidity
File: /liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
184:    function distributeToAllHolders() public {
185:        uint256 num = holders.length;
186:        if (num > 0) {
187:            distribute(holders.length);
188:        }
189:    }
```

Calling `distribute(holders.length)` forces us to do unnecessary operations.
The implementation of `distribute()`  has a line that checks for the minimum value(Limit) as shown below
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198-L211
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
```

Passing `holders.length` as the function argument means our `Math.min()` is evaluated as follows

```solidity
        uint256 limit = Math.min(
            nextDistributionRecipient + holders.length,
            holders.length
        );
```

Since `nextDistributionRecipient` is of type `uint` the lowest value it can have is `0` meaning we get 
```solidity
        uint256 limit = Math.min(
            0 + holders.length,
            holders.length
        );
```
This makes the only possible result to be `holders.length` being the minimum value thus no need to even check

```diff
@@ -184,7 +216,40 @@ contract LiquidInfrastructureERC20 is
     function distributeToAllHolders() public {
         uint256 num = holders.length;
         if (num > 0) {
-            distribute(holders.length);
+            if (!LockedForDistribution) {
+                require(
+                    _isPastMinDistributionPeriod(),
+                    "MinDistributionPeriod not met"
+                );
+                _beginDistribution();
+            }
+
+            uint i;
+            for (i = nextDistributionRecipient; i < num; i++) {
+                address recipient = holders[i];
+                if (isApprovedHolder(recipient)) {
+                    uint256[] memory receipts = new uint256[](
+                        distributableERC20s.length
+                    );
+                    for (uint j = 0; j < distributableERC20s.length; j++) {
+                        IERC20 toDistribute = IERC20(distributableERC20s[j]);
+                        uint256 entitlement = erc20EntitlementPerUnit[j] *
+                            this.balanceOf(recipient);
+                        if (toDistribute.transfer(recipient, entitlement)) {
+                            receipts[j] = entitlement;
+                        }
+                    }
+
+                    emit Distribution(recipient, distributableERC20s, receipts);
+                }
+            }
+            nextDistributionRecipient = i;
+
+            if (nextDistributionRecipient == num) {
+                _endDistribution();
+            }
         }
     }

```

In the diff above, we got rid of the entire line calculating the limit, and we utilize the `holders.length` that was cached in the variable `num`
We also don't need the check `require(numDistributions > 0, "must process at least 1 distribution");` anymore as we have the `if` check ensuring we are only dealing with value greater than 0 inside the block


**To get consistent gas results , the tests were modified to always have same list of revenue instead of having randoms for the sake of gas benchmarks**
```diff
diff --git a/liquid-infrastructure/test/liquidERC20.ts b/liquid-infrastructure/test/liquidERC20.ts
index 0121500..d422bac 100644
--- a/liquid-infrastructure/test/liquidERC20.ts
+++ b/liquid-infrastructure/test/liquidERC20.ts
@@ -356,10 +356,14 @@ async function randomDistributionTests(
   const signers = await ethers.getSigners();
   const holderAllocations = randomDivisions(supply, BigInt(numAccounts));
   const totalRevenue = supply * BigInt(20);
-  const revenue = randomDivisions(
-    totalRevenue,
-    BigInt(Math.ceil(Math.random() * 15))
-  ); // Random distributions of lots of rewards
+  const revenue =  [
+    373116135360165335531520n,
+    3127510666889880817106944n,
+    3843416596856115889700864n,
+    11776761760536179328942080n,
+    319198751103732938702848n,
+    559996089253925690015744n
+  ]; // Random distributions of lots of rewards
   if (holderAllocations == null) {
     throw new Error("Unable to generate random divisions");
```

## Unnecessary checks being performed due to the arguement passed(Save 621 Gas on average)
**Gas before and after**
|        | Min  | Max | AVG | 
| ------ | ---- | ------- | ------ |
| Before | 30369 | 117553 | 95757 |
| After  | 29748 | 116932  |95136 |

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L351-L353
```solidity
File: /liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
351:    function withdrawFromAllManagedNFTs() public {
352:        withdrawFromManagedNFTs(ManagedNFTs.length);
353:    }
```

Calling the function `withdrawFromManagedNFTs` with the argument `ManagedNFTs.length` results in us making redundant checks
In the implementation of `withdrawFromManagedNFTs` we have the following
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359-L386
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

    }
```

The important snippet is shown below 
```solidity
        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
```

Passing the variable `ManagedNFTs.length` as the argument  `numWithdrawals`  (`withdrawFromManagedNFTs(ManagedNFTs.length)`)results in the following

```solidity
        uint256 limit = Math.min(
            ManagedNFTs.length + nextWithdrawal,
            ManagedNFTs.length
        );
```
Regardless of what the value of `nextWithdrawal` is, the minimum value will always be `ManagedNFTs.length` since the lowest value `nextWithdrawal` can take is `0` 
We therefore do not require this check if we are passing `ManagedNFTs.length` as the argument


```diff
@@ -349,7 +381,29 @@ contract LiquidInfrastructureERC20 is
     }

     function withdrawFromAllManagedNFTs() public {
-        withdrawFromManagedNFTs(ManagedNFTs.length);
+        require(!LockedForDistribution, "cannot withdraw during distribution");
+
+        if (nextWithdrawal == 0) {
+            emit WithdrawalStarted();
+        }
+
+        uint256 limit = ManagedNFTs.length
+        uint256 i;
+        for (i = nextWithdrawal; i < limit; i++) {
+            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
+                ManagedNFTs[i]
+            );
+
+            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
+            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
+            emit Withdrawal(address(withdrawFrom));
+        }
+        nextWithdrawal = i;
+
+        if (nextWithdrawal == limit) {
+            nextWithdrawal = 0;
+            emit WithdrawalFinished();
+        }
     }
```

Note, we inline to avoid messing with the other function. Also we now take advantage of the cached value  on line `uint256 limit = ManagedNFTs.length`

## Pack `lastDistribution` with `lockedForDistribution`(Save 1 SLOT: 2000 Gas) 
**type `uint128` should be big enough to hold block numbers**
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L70-L80
```solidity
File: /liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
70:    uint256 public LastDistribution;

75:    uint256 public MinDistributionPeriod;

80:    bool public LockedForDistribution;
```

`lastDistribution` holds the block number thus we can reduce the size to `uint128` which should be big enough

```diff
diff --git a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
index 4722279..a87e229 100644
--- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
+++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
@@ -67,17 +67,17 @@ contract LiquidInfrastructureERC20 is
     /**
      * @notice Holds the block of the last distribution, used for limiting distribution lock ups
      */
-    uint256 public LastDistribution;
+    uint128 public LastDistribution; //@audit a block , maybe a smaller type for this and pack with bool?

     /**
-     * @notice Holds the minimum number of blocks required to elapse before a new distribution can begin
+     * @notice When true, locks all transfers, mints, and burns until the current distribution has completed
      */
-    uint256 public MinDistributionPeriod;
+    bool public LockedForDistribution;

     /**
-     * @notice When true, locks all transfers, mints, and burns until the current distribution has completed
+     * @notice Holds the minimum number of blocks required to elapse before a new distribution can begin
      */
-    bool public LockedForDistribution;
+    uint256 public MinDistributionPeriod;

```



## Conclusion
It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.









