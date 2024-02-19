[G-01]- Don’t make variables public unless necessary
Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be 

public.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L46

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L54

[G-02]-Don’t cache if used only once.
Caching here will simply increase gas cost, since it doesn’t affect readability, we should not cache since we only reference the cached variables only once.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L140

Proposed Optimization - call directly ownerOf(account) inside the _withdrawBalancesTo(erc20s,ownerOf(account)) functions to save gas.

[G-03]-By following short circuiting strategy ordering lower cost operation first and then higher operation in && and || operator save gas.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L245

Proposed Optimization - in the following if statement  (totalSupply() == 0 || holders.length == 0)  the first operation totalSupply()0 is expensive and used more gas then second one holders.length0 the second on is cheaper it's samply read the length or array but the totalSupply() function may be complex function with lots of code and complex logic and need more gas by putting the second in first you can save gas becouse.

[G-04] use != instead of > for checking unsigned integer
erc20EntitlementPerUnit.length > 0 check Athe length or array erc20EntitlementPerUnit and the length of array is unsigned integer and this is cheaper to use !=0
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L265

Proposed Optimization - use !=0 instead of >0 to save gas like this (erc20EntitlementPerUnit.length != 0).