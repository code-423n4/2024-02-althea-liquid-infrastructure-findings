Based on this line of code 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L314-L323

mint function lacks a direct check for whether a distribution has occurred within the MinDistributionPeriod blocks.The mintAndDistribute function does check this before calling mint, but the mint function can be called independently so it requires this check also. 
consider adding require statement at the beginning of the mint function to check whether the minimum distribution period has passed.