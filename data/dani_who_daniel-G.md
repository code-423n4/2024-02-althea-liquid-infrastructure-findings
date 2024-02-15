### LiquidInfrastructureNFT multiple inheritance, possible diamond problem (LiquidInfrastructureNFT.sol line 33)

`LiquidInfrastructureNFT` inherits from `ERC721` and `OwnableApprovableERC721`, which also inherits from `ERC721`. As  `LiquidInfrastructureNFT`not upgradeable and there is no `ERC721` overloaded function in `OwnableApprovableERC721` it is not a security risk but the deployment cost is higher. Safe to remove `ERC721` inheritance from `LiquidInfrastructureNFT`.

### No need to use intermediary `address erc20` variable in `_withdrawBalancesTo()` (LiquidInfrastructureNFT.sol line 176)
Intermediary variable is not necessary, costs extra. Best to use calldata var `erc20s[i]` directly.

### No need to use intermediary `address destination` variable in `withdrawBalances()` (LiquidInfrastructureNFT.sol line 141)
Intermediary variable is not necessary, costs extra. Best to use `ownerOf(AccountId)` directly at function invocation.

### No need to use intermediary `bool stillHolding` variable in `_afterTokenTransfer()` (LiquidInfrastructureERC20.sol line 169)
Intermediary variable is not necessary, costs extra. Best to use `this.balanceOf(from) != 0` directly in `if` clause.

### No need to use intermediary `uint256 num` variable in `distributeToAllHolders()` (LiquidInfrastructureERC20.sol line 185)
Intermediary variable is not necessary, costs extra. Best to use `holders.length` directly in `if` clause.

### LastDistribution and MinDistributionPeriod could be on same slot by reducing size to 128 bits
Also since they are used together, can be grouped together e.g.: into a struct, which then can be accessed (SLOAD) once for example in _isPastMinDistributionPeriod. uint128 would be enough hence it's max value is 340282366920938463463374607431768211455. (Of course when used, `block.timestamp` might need to be casted too in some cases, depending on the usage.)