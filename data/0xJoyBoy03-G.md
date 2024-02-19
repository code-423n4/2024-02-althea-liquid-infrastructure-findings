Summary:
 Every time the owner wants to mint for others, there's a wasteful gas consumption in every mint process.

Detail:
 in the `_afterTokenTransfer` function :
```js
function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        // @audit-gas : add this condition to the if statement to don't query over zero address to save gas: if(!stillHolding && from != address(0))
        if (!stillHolding) {
            for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }
        }
    }
```


every time the owner mints for the approved user, there is a loop in this function that checks if this `from` address has any balance or not but the zero address's balance is always zero. By checking if `from` is not a zero address `(from != address(0))`, unnecessary gas consumption can be prevented, as querying over a zero address is redundant and wasteful.


Recommendation:
Add a condition to the if statement to avoid querying over a zero address

```diff
// existing codes...

+  if(!stillHolding && from != address(0)) {
// existing codes
}
```