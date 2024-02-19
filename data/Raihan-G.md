# GAS OPTIMIZATION

# SUMMARY
|      |  issue  |
|------|---------|
|[G‑01]|Gas Optimization Strategy for _beginDistribution Function|
|[G‑02]|Gas Optimization: Declare Variables Outside the Loop|
|[G‑03]|Access mappings directly rather than using accessor functions|
|[G‑04]|Delete variables that you don't need|
|[G‑05]|Uncheck arithmetics operations that can’t underflow/overflow|
|[G‑06]|Use Modifiers Instead of Functions To Save Gas|
|[G‑07]|Cache external calls outside of loop to avoid re-calling function on each iteration|
|[G‑08]|Pre-increment and pre-decrement are cheaper thanĀ +1 ,-1|
|[G‑09]|It is sometimes cheaper to cache calldata|
|[G‑10]|Using XOR (^) and AND (&) bitwise equivalents|
|[G‑11]|`Calldata` pointer is used instead of `memory` pointer for loops|

## [G-01] Gas Optimization Strategy for _beginDistribution Function

**Description:**
In the `LiquidInfrastructureERC20` contract, the `LockedForDistribution` state variable is utilized in both the `_beginDistribution` and `distribute` functions. While one read occurs inside the `_beginDistribution` function, the `distribute` function performs two reads—one for the inline function and one for its own usage. Each storage read operation consumes 100 gas, posing a significant gas cost concern.
[_beginDistribution](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L257-L281)
[distribute](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L205)

### Recommendation:
To optimize gas usage in the `LiquidInfrastructureERC20` contract, we propose the following steps:

1. **Inline _beginDistribution Function:** Merge the logic of the `_beginDistribution` function into the `distribute` function to avoid an additional storage read.

2. **Cache LockedForDistribution Storage:** Store the value of `LockedForDistribution` in a local variable within the `distribute` function to avoid repeated storage reads.

## [G-02] Gas Optimization: Declare Variables Outside the Loop

**Explanation of Issue:**
When variables are declared inside a loop in Solidity, a new instance of the variable is created for each iteration of the loop. This can result in unnecessary gas costs, especially when the loop is executed frequently or iterates over a large number of elements. To optimize gas usage, it is advisable to declare the variable outside the loop and initialize it to zero to avoid the creation of multiple instances.

**Examples of Code:**

- *Non-optimized Code:*
```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        for (uint256 i = 0; i < values.length; i++) {
            uint256 total; // This declaration is inside the loop and not initialized
            total += values[i];
        }
        
        return total; // This line will result in a compilation error; total is not visible here
    }
}
```

- *Optimized Code:*
```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total; // Declare the variable outside the loop without initializing
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

**Reason:**
Declaring the variable outside the loop without initializing it inside the declaration is still an improvement, as it avoids the creation of multiple instances of the variable during each iteration. However, initializing the variable to zero inside the declaration further enhances gas optimization. This is crucial, especially in scenarios where the loop is executed frequently or processes a substantial number of elements, contributing to more efficient and cost-effective smart contracts.

```solidity
215   address recipient = holders[i];

275   uint256 entitlement = balance / supply;

422   address managed = ManagedNFTs[i];
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L215



```solidity
176      address erc20 = erc20s[i];
177      uint256 balance = IERC20(erc20).balanceOf(address(this));
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L176-L177

## [G-03] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup

```solidity
97  return HolderAllowlist[account];
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L97

## [G-04] Delete variables that you don't need
In Ethereum, you get a gas refund for freeing up storage space.
Deleting a variable refund 15,000 gas up to a maximum of half the gas cost of the transaction. Deleting with the delete keyword is equivalent to assigning the initial value for the data type, such as 0 for integers.

There are 1 instances of this issue:

```solidity
266  delete erc20EntitlementPerUnit;

291  delete erc20EntitlementPerUnit;
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L266

```solidity
217        delete thresholdErc20s;
218        delete thresholdAmounts;
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L217-L218

## [G-05] Uncheck arithmetics operations that can’t underflow/overflow
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block



```solidity
249  return (block.number - LastDistribution) >= MinDistributionPeriod;

275   uint256 entitlement = balance / supply;
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L249

## [G-06] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.
There are 1 instances of this issue:


```solidity
243  function _isPastMinDistributionPeriod() internal view returns (bool) {
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L243

## [G-07] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.



```solidity
177  uint256 balance = IERC20(erc20).balanceOf(address(this));

179  bool result = IERC20(erc20).transfer(destination, balance);
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L177

## [G-08] Pre-increment and pre-decrement are cheaper thanĀ +1 ,-1

```solidity
174  holders[i] = holders[holders.length - 1];

225  ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L174

## [G-09] It is sometimes cheaper to cache calldata
Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.

```solidity
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}
```


- these are use in loop cache it in local to save gas
```solidity
109  address[] calldata newErc20s,

171  address[] calldata erc20s,
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L109

## [G-10] Using XOR (^) and AND (&) bitwise equivalents
Given 4 variables a, b, c and d represented as such:

```
0 0 0 0 0 1 1 0 <- a
0 1 1 0 0 1 1 0 <- b
0 0 0 0 0 0 0 0 <- c
1 1 1 1 1 1 1 1 <- d

```


To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there‚Äôs at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

```solidity
139  if (from == address(0) || to == address(0)) {

245  if (totalSupply() == 0 || holders.length == 0) {    
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L139

## [G-11] `Calldata` pointer is used instead of `memory` pointer for loops
If our function parameter is specified as `calldata`, using a memory pointer will copy that `calldata` value into memory, which can result in memory expansion costs. To avoid this, we should use a calldata pointer to read from `calldata` directly.

```
- Memory copy: 20 gas
- Calldata read: 3 ga
```

```solidity
217  uint256[] memory receipts = new uint256[](    
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L217

```solidity
174  uint256[] memory amounts = new uint256[](erc20s.length);
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L174