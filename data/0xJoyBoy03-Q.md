Summary:
everytime the approved users burn their tokens, the zero address will be pushed to the holders array.

Detail:
inside the `_beforeTokenTransfer` function:

```js
function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
            require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        // @note: Alice and users that Alice approved can burn Alice's tokens to push address(0) to `holders`
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
            holders.push(to);
        }
    }
```

as you see if approved users burn their tokens, the zero address will be pushed to the `holders` array cause the zero address's balance is always 0.
this is because, in the ERC20.sol in the `_burn` function, the function reduces the balance of the `from` but doesn't increase the balance of the zero address.
so It will cause to `exists` variable to always be true if `to` is a zero address.


Impact:
Holders should be actual holders not a bunch of zero addresses. this is just breaking the functionality of the protocol.

Recommendation:

add another condition in the if statement to don't push zero addresses:
```diff

//existing codes...

+    if(!exists && to != address(0))

//existing codes...
```