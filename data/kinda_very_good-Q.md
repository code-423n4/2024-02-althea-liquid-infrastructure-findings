```
  function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s; 
        There is no check to ensure the addresses implment the IERC20 standard, this could cause a panic when functions attempt to implement an instance of IERC20 at the address
    }
This can however easily be solved by the owner setting a new distributableERC20s array
```

```
There is no logic to prevent the owner from adding the same contract multiple times to the managedNFTS array 
```