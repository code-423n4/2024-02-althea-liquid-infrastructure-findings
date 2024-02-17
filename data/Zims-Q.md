Lockin the value `LockedForDistribution` in `LiquidInfrastructureERC20` could brick the whole contract.

Altho I cant identify a way to do it. It might be a good security step to add a onlyAdmin function that could manually unlock the variable in cases someone manages to DOS the `distribute` and `_endDistribution` is never called so `LockedForDistribution` stays true forever.
```
function setLockedForDistribution(bool _lockedForDistribution) public onlyOwner{
        LockedForDistribution = _lockedForDistribution;
    }
```