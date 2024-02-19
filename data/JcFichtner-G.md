## [G-01] Block numbers in storage do not need to be uint256

A block number of size uint48 will work for millions of years into the future. A block number increments once every 12 seconds. This should give you a sense of the size of numbers that are sensible.

> uint256 public LastDistribution;

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L70