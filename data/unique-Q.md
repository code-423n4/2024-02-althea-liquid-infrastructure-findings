## \[L-01\] Insufficient coverage

Description  
The test coverage rate of the project is ~95%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

&nbsp;

## \[N-02\] No Need to cache `this.totalSupply();`

```
270: uint256 supply = this.totalSupply();
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L270

&nbsp;

## \[N-03\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-02\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N-04\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-05\] Function writing that does not comply with the Solidity Style Guide

Context  
All Contracts

Description  
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor  
receive function (if exists)  
fallback function (if exists)  
external  
public  
internal  
private  
within a grouping, place the view and pure functions last

## \[S-01\] Generate perfect code headers every time

Description:  
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## \[S-02\] You can explain the operation of critical functions in NatSpec with an infographic.