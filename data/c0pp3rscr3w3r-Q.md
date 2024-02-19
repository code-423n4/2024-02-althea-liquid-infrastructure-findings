## Emit events
emit events at important transactions like approving a holder, and disapproving
```solidity
    function approveHolder(address holder) public onlyOwner {
        require(!isApprovedHolder(holder), "holder already approved");
        HolderAllowlist[holder] = true;
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106-L109

## _mint can be called instead of the public mint
```solidity
    function mintAndDistribute(
        address account,
        uint256 amount
    ) public onlyOwner {
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
        }
        -> mint(account, amount); @audit
    }


    function mint(
        address account,
        uint256 amount
    ) public onlyOwner nonReentrant {
        _mint(account, amount);
    }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L310