Solidity array assignment can be used to save extra gas.

Instead of manually delete the original array and then looping, use solidity provided array assginment to save gas:

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L117-L123

use :

```solidity 
thresholdErc20s = newErc20s;
thresholdAmounts=newAmounts;
```