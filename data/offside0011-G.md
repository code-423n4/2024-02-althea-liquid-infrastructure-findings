Solidity array assignment can be used to save extra gas.

Instead of manually delete the original array and then looping, use solidity provided array assginment to save gas:

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L117-L123

```solidity
delete thresholdErc20s;
delete thresholdAmounts;

for (uint i = 0; i < newErc20s.length; i++) {
    thresholdErc20s.push(newErc20s[i]);
    thresholdAmounts.push(newAmounts[i]);
}
```

use :

```solidity 
thresholdErc20s = newErc20s;
thresholdAmounts=newAmounts;
```