In Liquid Infrastructure ERC20.sol contract, in distributionAl Holders() function can remove num variable and put holders.length into the brackets to save gas

-Before-
```
function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
              distribute(holders.length);
        }
    }
```

-After-
```
function distributeToAllHolders() public {
        if (holders.length > 0) {
              distribute(holders.length);
        }
    }
```

