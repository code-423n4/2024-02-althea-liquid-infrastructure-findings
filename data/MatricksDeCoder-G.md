### GAS-1 Remove unused Maths library =>  unused constructs lead to increased code size for no reason which increases deployment costs 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L4 

### GAS-2 Use Solmate Libraries instead of OpenZeppelin libraries => Solmate libraries are more gas optimized than OpenZeppelin libraries which can lead to gas savings for deployments and function calls 

### GAS-3 Use Assembly to emit events => can use assembly to emit events efficiently by utilizing scratch space and the free memory pointer. This will allow us to potentially avoid memory expansion costs. 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L229

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L280

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L229

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L363

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L433

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L124

### GAS-4 Use do while loops instead of for loops => A do while loop will cost less gas since the condition is not being checked for the first iteration, 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L271

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L120


