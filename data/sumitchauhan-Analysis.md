Medium Issues

1.Reentrancy:
LiquidInfrastructureERC20.sol::378=> emit Withdrawal(address(withdrawFrom));

Low Issues

1.Local Variable Shadowing:
LiquidInfrastructureERC20.sol::463=> constructor(string,string,address[],address[],uint256,address[])._name shadows: - ERC20._name (state variable)
LiquidInfrastructureERC20.sol::376=> constructor(string,string,address[],address[],uint256,address[])._symbol shadows: - ERC20._symbol (state variable)

2.External Calls inside a loop:
LiquidInfrastructureERC20.sol::222=> distribute(uint256) has external calls inside a loop: entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient)
LiquidInfrastructureERC20.sol::222=> withdrawFromManagedNFTs(uint256) has external calls inside a loop: (withdrawERC20s) = withdrawFrom.getThresholds() 

Gas Optimisation

1.Cache Array Length
LiquidInfrastructureERC20.sol::218=> Loop condition j < distributableERC20s.length should use cached array length instead of referencing `length` member of the storage array.

2.Variables should be declared Immutable:
LiquidInfrastructureERC20.sol::75=> MinDistributionPeriod should be immutable 

3.Public variable read in external context
LiquidInfrastructureERC20.sol::142=> _beforeTokenTransfer(address,address,uint256) reads exists = (this.balanceOf(to) != 0) with `this` which adds an extra STATICCALL.
LiquidInfrastructureERC20.sol::223=> distribute(uint256) reads entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient) with `this` which adds an extra STATICCALL.
LiquidInfrastructureERC20.sol::270=> _beginDistribution() reads supply = this.totalSupply() with `this` which adds an extra STATICCALL.


### Time spent:
18 hours