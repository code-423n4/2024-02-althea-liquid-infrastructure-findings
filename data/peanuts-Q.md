### [L-01] The burn address will be part of the holders array

When burning tokens, the `to` address will be address(0). This `to` address will be appended to the `holders` array during `burn()` as `_beforeTokenTransfer()` is invoked, which is

```
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
>           holders.push(to);
        }
```

Have a clause that disallows the burn address to be part of the holders array.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L138-L145

### [L-02] `MinDistributionPeriod` cannot be changed

Once `MinDistributionPeriod` is set in the constructor, it cannot be changed by anyone. This is an issue because the block.number of different chains may differ in the future due to upgrades, and to accommodate to the changes, `MinDistributionPeriod` should be flexible.

```
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
        ManagedNFTs = _managedNFTs;
        ...

>       MinDistributionPeriod = _minDistributionPeriod;
```

Create a function that allows the owner to change `MinDistributionPeriod`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L471

### [L-03] If `MinDistributionPeriod` is set at 0, then minting and burning can happen before distribution.

The README states:

> mint() and burn() should not work when the minimum distribution period has elapsed.

If there is no minimum distribution period, then `mint()` and `burn()` can work when the minimum distribution period has elapsed.

Enforce that MinDistributionPeriod cannot be a zero value in the constructor.

```
        for (uint i = 0; i < _approvedHolders.length; i++) {
            HolderAllowlist[_approvedHolders[i]] = true;
        }

        require(MinDistributionPeriod != 0, "MinDistributionPeriod is set at 0");
        MinDistributionPeriod = _minDistributionPeriod;

        
        distributableERC20s = _distributableErc20s;
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L151-L156

### [L-04] Users cannot transfer Althea tokens even if they get approval

The `_beforeTokenTransfer()` function is overriden from the base ERC20 contract, and so if a user transfers to another user via a `transferFrom()` function, the transfer will not work. The approvee needs to be an approved holder as well.

```
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
>           require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
    }
```

Since the transfer does not work, only the `burnFrom()` function works. I'm not sure about the exact purpose of burning the token since all it does is create extra attack vectors. As such, the approval in ERC20 should be overridden completely and should not be in use.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127-L146

### [L-05] `distribute()` may reach out of gas error if the length of the arrays of approved holders and approved ERC20s is too large

If there is a lot of approved holders and a lot of ERC20 type tokens to distribute, then the function `distribute()` will continuously face out of gas errors. The main problem is the nested loop structure, which will consume a lot of gas.

```
        uint i;
        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L213-L230

### [L-06] Althea token will be stuck in the contract if the holder is disapproved

When the holder is disapproved by the owner, the `HolderAllowlist[holder]` will be false.

```
    function disapproveHolder(address holder) public onlyOwner {
        require(isApprovedHolder(holder), "holder not approved");
        HolderAllowlist[holder] = false;
    }
```

The holder cannot transfer their tokens out as well, because of the overridden `_transferFrom()` function. The Althea token will be stuck in the contract.

Consider burning the holder's tokens if the owner decides to disapprove their token.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116-L120

### [L-07] Holders can grief the distribution process through blacklisted tokens if the reward tokens is in USDC

It is known that the reward token for being a holder of LiquidInfrastructureERC20 token is USDC. Users that holds the LiquidInfrastructureERC20 token will get some USDC rewards when `distribute()` is called.

```
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
>                   if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
```

A malicious holder can grief the distribute function by blacklisting themselves so that they cannot receive USDC tokens. As a result, the whole function will fail since the transaction is atomic.

The whole distribution process will fail. Distribution caller will waste money on gas.

Note that some other ERC20s also have a blacklist option. Take note of this when accepting ERC20 tokens as reward. If possible, break the function apart so that is it not atomic and if someone tries to grief the transaction, it will not work.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L219-L226

### [L-08] Some tokens revert on zero-value transfer

When distributing tokens, the distribute function does not take into account that the distributed sum can be zero if there is no revenue generated from the NFTs.

```
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
>                   if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
```

Some ERC20 tokens, eg LEND, will revert on zero value transfers. Take note of those tokens before adding it to the revenue.

https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L446