`Distribute` function don't distribute all rewards to holders in case when not approved addresses still have LiquidInffrastractureERC20 tokens. In that case their portions of rewards remain in LiquidInffrastractureERC20 contract. So new added holders will get some extra reward in next distribution instead of previous holders.

After burning LiquidInffrastractureERC20 tokens zero address is pushed into `address[] private holders`.

Additional instances of an issue listed in the automated findings report:

[Access control ](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md#m-02-burning-is-missing-access-control) => [burnAndDistribute](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L331)

[Unsafe transfer() and transferFrom()](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md#m-03-unsafe-use-of-transfertransferfrom-with-ierc20) => [transfer](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224)