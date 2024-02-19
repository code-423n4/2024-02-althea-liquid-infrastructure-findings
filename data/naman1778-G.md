## [G-01] Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

```
File: LiquidInfrastructureERC20.sol	

139: if (from == address(0) || to == address(0)) {

245: if (totalSupply() == 0 || holders.length == 0) {
```
    /liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
    index 4722279..bba6da9 100644
    --- a/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
    +++ b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
    @@ -136,7 +136,7 @@ contract LiquidInfrastructureERC20 is
                     "receiver not approved to hold the token"
                 );
             }
    -        if (from == address(0) || to == address(0)) {
    +        if (((from ^ address(0)) & (to ^ address(0))) == 0) {
                 _beforeMintOrBurn();
             }
             bool exists = (this.balanceOf(to) != 0);
    @@ -242,7 +242,7 @@ contract LiquidInfrastructureERC20 is
          */
         function _isPastMinDistributionPeriod() internal view returns (bool) {
             // Do not force a distribution with no holders or supply
    -        if (totalSupply() == 0 || holders.length == 0) {
    +        if (((totalSupply() ^ 0) & (holders.length ^ 0)) == 0) {
                 return false;
             }

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |

| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |