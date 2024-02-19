QA Report

Issue #1

The version of solidity currently used is not the latest compiler version.

```
pragma solidity 0.8.12;
```
This version has a bug that affects the protocols code base.

SOL-2022-5

!!!External links to solidity official docs and blog
https://docs.soliditylang.org/en/latest/bugs.html
https://blog.soliditylang.org/2022/06/15/dirty-bytes-array-to-storage-bug/

Solution 

Update pragma statements to latest version

```
pragma solidity ^0.8.20;
```

Issue #2

Lack of hard limit on [holders](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48) and [ERC20](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L46) distribution arrays makes the protocol vulnerable to possible DOS attacks.

The protocol in essence trusts the owner to mitigate DOS attacks, and this should not be allowed as I understand in the design of the protocol owners are not fully trusted.

Consider the [loop](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214) in the distribution function

```
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
``` 

Since the protocol theoretically allows an unlimited number of distributable tokens, this could theoretically be out of service due to a single user distribution.

Solution

I would recommend a hard limit on the number of token distribution arrays and holders array be set. For example

```
function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        if (_distributableERC20s.length > 50) {
            revert("Token Count Exceeds Limit");
        };
        distributableERC20s = _distributableERC20s;
    }
```

Issue #3

The protocol has no fail-safe mechanism in case of locked funds, DOS attacks or emergent problems which can lock funds permanently in the LiquidInfrastructureERC20.sol contract, of-course such solutions may seem like a matter of preference, However since this part of the protocol has not been battle tested, I would recommend that such fail safes be considered.

Solution

I would recommend liquidity extraction functions be added, that may be renounced at a later point in time, or an emergency mode in which all contract functions are paused to mitigate further damage while a solution is being made. 

### Time spent:
10 hours