#### NC-1 Only constant or immutable variables should start with Capital Letters or be capitalized else use camelCase

-Description => Some code instances capitalize variables like e.g  bool public LockedForDistribution;uint256 public MinDistributionPeriod; etc

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L60 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L65

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L70

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L75

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L80

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L53

-Impact => This can lead to problems with code readability, consistency
-Recommendation => It is recommended to use camelCase for all other variables that are not constant or immutable 

### NC-2 Spelling and grammar errors

-Description => There are instances of typos, misspellings, unclear grammar, mistakes, etc in the code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L17
Wrong spelling -> LiquidInfrastructreNFTs. (Infrastructure vs Infrastructre)

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L28
Wrong spelling -> Occassionally vs Occasionally

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L100
Wrong spelling -> addressses vs addresses

-Impact => This can lead to problems with code readability, auditing, maintaining, or even errors if the dev misinterprets comments
-Recommendation => It is recommended to fix all typos, spelling and grammar errors

#### NC-3 Some if-else statement can be converted to a ternary

-Description => Some if-else statement can be changed to ternary for example here the is-else in the modifier link below 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L39

-Impact => Improving code readability and compactness is an integral part of optimal programming practices. The use of ternary operators in place of if-else conditions is one such measure. Ternary operators allow us to write conditional statements more concisely, thereby enhancing readability and simplicity. 
-Recommendation => It is recommended to use ternary in form => condition ? exprIfTrue : exprIfFalse

#### NC-4 Unused or unnecessary Maths library import 

Description -> LiquidInfrastructureERC20.sol imports Maths library which is never used 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L4

Impact => Impacts code readability, size, gas costs, maintainability 
-Recommendation => Remove the Maths library import 



