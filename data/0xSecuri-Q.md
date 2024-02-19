### Low Risk

| Count  | Title                                      | Context |
| :----: | :----------------------------------------- | ------- |
| [L-01] | Missing events for important state changes | 2       |
| [L-02] | Emitting events with improper data         | 1       |

| Total Low Risk Issues |  2  |
| :-------------------: | :-: |

### Non-Critical

|  Count  | Title                                   | Context |
| :-----: | :-------------------------------------- | ------- |
| [NC-01] | Constant names do not follow convention | 3       |
| [NC-02] | Redundant require statement             | 1       |

| Total Non-Critical Issues |  2  |
| :-----------------------: | :-: |

## [L-01] Missing event emission for important state changes

## Impact

The protocol does not emit events when approving and disapproving holders for the protocol, which prevents the data from being observed easily by off-chain interfaces.

#### Examples

- [View the first code snippet in context](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106-L109)
- [View the second code snippet in context](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116-L119)

## Recommendation

Emit events for important state changes.

## [L-02] Emitting events with improper data

## Impact

At the following [line](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L229), the protocol emits the following `event: Distribution(recipient, distributableERC20s, receipts)`. Which should show how much was transfered to the user of his/her revenues. However, if in the [following line](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224-L226) `toDistribute` is a USDT token as it does not return any value in `transfer()`, we will never enter the if statement and therefore we will emit an event with wrong values - basically receipts will contain only 0 values.

## Recommendation

Consider implementing a mechanism which will fill `receipts` even when the protocol deals with tokens which do not return value on `transfer()`.

## [NC-01] Constant names do not follow convention

### Description

In the provided code snippets below, constant names such as `Version` and `AccountId` do not adhere to the Solidity's convention for constants.

#### Examples

[First issue](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L54) 
[Second issue](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L46) 
[Third issue](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L53) 

### Recommendation

Update the constant names to be all capital letters with underscores separating words to follow naming conventions. According to the [Solidity documentation](https://docs.soliditylang.org/en/v0.8.12/style-guide.html#constants), constant names should be all capital letters with underscores separating words.

## [NC-02] Redundant require statement

### Description

At the following [line](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431), there is a `require()` statement which is always true. Probably it was added mistakenly or was part of the previous implementation, but it seems redundant.

### Recommendation

Consider removing the redundant `require()` statement
