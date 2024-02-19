# Low Risk

## L-1: Centralization Risk for trusted owners

Contracts have owners with privileged rights to perform admin tasks and need to be trusted to not perform malicious updates or drain funds.

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 29](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L29)

	```solidity
	contract LiquidInfrastructureERC20 is
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 106](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106)

	```solidity
	    function approveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 116](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116)

	```solidity
	    function disapproveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 306](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L306)

	```solidity
	    ) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 321](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L321)

	```solidity
	    ) public onlyOwner nonReentrant {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 394](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394)

	```solidity
	    function addManagedNFT(address nftContract) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 416](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L416)

	```solidity
	    ) public onlyOwner nonReentrant {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 443](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L443)

	```solidity
	    ) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 203](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L203)

	```solidity
	    function recoverAccount() public virtual onlyOwner(AccountId) {
	```

## L-2: Unsafe ERC20 Operations should not be used

ERC20 functions may not behave as expected. For example: return values are not always meaningful. It is recommended to use OpenZeppelin's SafeERC20 library.

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 224](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224)

	```solidity
	                    if (toDistribute.transfer(recipient, entitlement)) {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 418](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L418)

	```solidity
	        nft.transferFrom(address(this), to, nft.AccountId());
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 179](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L179)

	```solidity
	                bool result = IERC20(erc20).transfer(destination, balance);
	```




## L-3: Missing checks for `address(0)` when assigning values to address state variables

Assigning values to address state variables without checking for `address(0)`.

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 444](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L444)

	```solidity
	        distributableERC20s = _distributableERC20s;
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 464](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L464)

	```solidity
	        ManagedNFTs = _managedNFTs;
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 473](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L473)

	```solidity
	        distributableERC20s = _distributableErc20s;
	```



## L-4: Functions not used internally could be marked external

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 106](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106)

	```solidity
	    function approveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 116](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116)

	```solidity
	    function disapproveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 303](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303)

	```solidity
	    function mintAndDistribute(
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 331](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L331)

	```solidity
	    function burnAndDistribute(uint256 amount) public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 344](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L344)

	```solidity
	    function burnFromAndDistribute(address account, uint256 amount) public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 351](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L351)

	```solidity
	    function withdrawFromAllManagedNFTs() public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 394](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394)

	```solidity
	    function addManagedNFT(address nftContract) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 413](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413)

	```solidity
	    function releaseManagedNFT(
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 441](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441)

	```solidity
	    function setDistributableERC20s(
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 84](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L84)

	```solidity
	    function getThresholds()
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 108](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108)

	```solidity
	    function setThresholds(
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 135](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L135)

	```solidity
	    function withdrawBalances(address[] calldata erc20s) public virtual {
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 153](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L153)

	```solidity
	    function withdrawBalancesTo(
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 203](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L203)

	```solidity
	    function recoverAccount() public virtual onlyOwner(AccountId) {
	```



## L-5: Constants should be defined and used instead of literals



- Found in contracts/LiquidInfrastructureERC20.sol [Line: 174](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L174)

	```solidity
	                    holders[i] = holders[holders.length - 1];
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 425](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L425)

	```solidity
	                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
	```


## L-6: Event is missing `indexed` fields

Index event fields make the field more quickly accessible to off-chain tools that parse events. However, note that each index field costs extra gas during emission, so it's not necessarily best to index the maximum allowed per event (three fields). Each event should use three indexed fields if there are three or more fields, and gas usage is not particularly of concern for the events in question. If there are fewer than three fields, all of the fields should be indexed.

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 38](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L38)

	```solidity
	    event Distribution(address recipient, address[] tokens, uint256[] amounts);
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 41](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L41)

	```solidity
	    event Withdrawal(address source);
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 43](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L43)

	```solidity
	    event AddManagedNFT(address nft);
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 44](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L44)

	```solidity
	    event ReleaseManagedNFT(address nft, address to);
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 34](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L34)

	```solidity
	    event SuccessfulWithdrawal(address to, address[] erc20s, uint256[] amounts);
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 36](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L36)

	```solidity
	    event SuccessfulRecovery(address[] erc20s, uint256[] amounts);
	```

- Found in contracts/LiquidInfrastructureNFT.sol [Line: 37](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L37)

	```solidity
	    event ThresholdsChanged(address[] newErc20s, uint256[] newAmounts);
	```



## L-7: The `nonReentrant` `modifier` should occur before all other modifiers

This is a best-practice to protect against reentrancy in other modifiers

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 321](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L321)

	```solidity
	    ) public onlyOwner nonReentrant {
	```

- Found in contracts/LiquidInfrastructureERC20.sol [Line: 416](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L416)

	```solidity
	    ) public onlyOwner nonReentrant {
	```
