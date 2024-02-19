## \[G‑01\] Save gas by preventing zero amount in  burn() and mint()

```
_mint(account, amount);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L322

&nbsp;

```
burn(amount);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L335

&nbsp;

## \[G-02\] Constants Variable Should Be Private for Gas Optimization

### Description

&nbsp;Using `private` for constant variables is cheaper in terms of gas usage. If the value of the constant variable is needed, it can be accessed via a getter function. In case of multiple constant variables, a single getter function could be used to return a tuple of the values of all currently-private constants.

```
46: uint256 public constant Version = 1;

53: uint256 public constant AccountId = 1;
```

&nbsp;https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L46

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L53

## \[G-03\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L398

&nbsp;https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L177

## \[G-04\] Use assembly to emit events

- Severity: Gas Optimization
- Confidence: High

### Description

This detector checks for instances where a contract uses assembly code to emit events. While it’s technically possible to emit events using inline assembly in Solidity, it’s generally discouraged due to readability concerns and potential for errors. Events should usually be emitted using higher-level Solidity constructs.

```
File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol


124: emit ThresholdsChanged(newErc20s, newAmounts);

184: emit SuccessfulWithdrawal(destination, erc20s, amounts);

204: emit TryRecover();
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L124

```
File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

363:             emit WithdrawalStarted();

378:            emit Withdrawal(address(withdrawFrom));

384:		 emit WithdrawalFinished();

402:            emit AddManagedNFT(nftContract);

433:        emit ReleaseManagedNFT(nftContract, to);

475:        emit Deployed();
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L363