## Summary:
 The function `LiquidInfrastructureERC20::_beforeTokenTransfer` is set as a `virtual` function. This function can be overridden by an inherited contract to change its behavior.

## PoC: 
The `LiquidInfrastructureERC20::_beforeTokenTransfer` function, when called should implement lock during distribution and add an address (token receiver) to the list of holders.
[line 127-131](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/2b26c04f0448505635210416bef9d3ce96391b16/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127-L131)



```javascript
    /**
     * Implements the lock during distributions, adds `to` to the list of holders when needed
     * @param from token sender
     * @param to  token receiver
     * @param amount  amount sent
     */
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
```

Since this function is set to virtual, it allows for an inherited contract to override the function and change its behavior. Also, the function should not be set to as override as it doesn't override any virtual function from parent/base contracts.
## Tools Used
Manual Review

## Impact: 
Since the function can be overridden, and its behavior changed any inherited contract can add an address to the list of holders unauthorized. Anyone can override the function and then add an address to the list of holders to receive tokens.

## Recommended Mitigation:
 Consider removing the virtual keyword from the `LiquidInfrastructureERC20::_beforeTokenTransfer`  function since it is not supposed to be overridden. The override keyword is unnecessary and should be removed. Also, access control should be implemented so that anyone should not be able to add any address to the list of holders to receive tokens.

```javascript
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal onlyOwner {
```