# [1] `_afterTokenTransfer()` is prone to DoS

**File:** `LiquidInfrastructureERC20.sol`

According to the scope, `DoS attacks against the LiquidInfrastructureERC20 are of significant concern`, thus we decided to include potential DoS attack scenario also in the QA report.

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L170)
```solidity
170:         if (!stillHolding) {
171:             for (uint i = 0; i < holders.length; i++) {
172:                 if (holders[i] == from) {
173:                     // Remove the element at i by copying the last one into its place and removing the last element
174:                     holders[i] = holders[holders.length - 1];
175:                     holders.pop();
176:                 }
```

Whenever `_afterTokenTransfer` is called, function iterates over `holders` mapping. If there are too many holders, function may revert with out of gas error - thus calling it won't be possible.

It's important to notice that this scenario might be possible, especially, because it's straightforwarded mentioned in the code-base.
Please take a look at `distributeToAllHolders()` NatSpec:

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L181)
```solidity
181:     /**
182:      * Performs a distribution to all of the current holders, which may trigger out of gas errors if there are too many holders
183:      */
184:     function distributeToAllHolders() public {
```

Since developers explicitly mentioned, that `distributeToAllHolders()` may trigger out of gas errors if there are too many holders - the same scenario may occur for  `_afterTokenTransfer()` - thus this function is also at risk of DoS.


# [2] Incorrect comment in `OwnableApprovableERC721.sol`

**File:** `OwnableApprovableERC721.sol`

[File: liquid-infrastructure/contracts/OwnableApprovableERC721.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L07)
```solidity
07: /**
08:  * An abstract contract which provides onlyOwner(id) and onlyOwnerOrApproved(id) modifiers derived from ERC721's
09:  * onwerOf, getApproved, and isApprovedForAll functions
10:  *
```

There is no `onwerOf` in ERC721. Above comment seems like a typo, it should be `ownerOf` instead of `onwerOf`.

# [3] Approving or disapproving holders do not emit any events

**File:** `LiquidInfrastructureERC20.sol`

It's a good practice to emit an event whenever some crucial operation is being performed, especially, when that operation updates the state of the blockchain.
Approving and disapproving holders do not implement event emission. It's recommended to emit events whenever those functions are called.

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106)
```solidity
106:     function approveHolder(address holder) public onlyOwner {
107:         require(!isApprovedHolder(holder), "holder already approved");
108:         HolderAllowlist[holder] = true;
109:     }
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L116)
```solidity
116:     function disapproveHolder(address holder) public onlyOwner {
117:         require(isApprovedHolder(holder), "holder not approved");
118:         HolderAllowlist[holder] = false;
119:     }
```

# [4] `MinDistributionPeriod` cannot be updated

**File:** `LiquidInfrastructureERC20.sol`

Variable `MinDistributionPeriod` is set only in the constructor and cannot be updated later.
Protocol does not implement any `onlyOwner` functions which allows to update this value in the future. Consider adding additional function (callable only by the owner, thus implementing `onlyOwner`) - which allows to either increase or decrease `MinDistributionPeriod`.

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L471)
```solidity
471:         MinDistributionPeriod = _minDistributionPeriod;
```


# [5] `setDistributableERC20s()` does not emit any event

**File:** `LiquidInfrastructureERC20.sol`

It's a good practice to emit an event whenever some crucial operation is being performed, especially, when that operation updates the state of the blockchain.
Function `setDistributableERC20s` updates the `distributableERC20s` state variable, however, it does not emit any event.

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441)
```solidity
441:     function setDistributableERC20s(
442:         address[] memory _distributableERC20s
443:     ) public onlyOwner {
444:         distributableERC20s = _distributableERC20s;
445:     }
```


# [6] Distributable ERC-20 list may contain duplicates

**File:** `LiquidInfrastructureERC20.sol`

The list of ERC20s which should be distributed from ManagedNFTs to holders is set either in `setDistributableERC20s()` or in `constructor()`. However, it's nowhere checked for duplicates.
It's possible to pass a list of the same addresses. 


[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441)
```solidity
441:     function setDistributableERC20s(
442:         address[] memory _distributableERC20s
443:     ) public onlyOwner {
444:         distributableERC20s = _distributableERC20s;
445:     }
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L456)
```solidity
456:     constructor(
457:         string memory _name,
458:         string memory _symbol,
459:         address[] memory _managedNFTs,
460:         address[] memory _approvedHolders,
461:         uint256 _minDistributionPeriod,
462:         address[] memory _distributableErc20s
463:     ) ERC20(_name, _symbol) Ownable() {
[...]
473:         distributableERC20s = _distributableErc20s;
```


# [7] Incorrect name of `setDistributableERC20s` function

**File:** `LiquidInfrastructureERC20.sol`

Since list of ERC20s which should be distributed from ManagedNFTs is already set in the `constructor`, function `setDistributableERC20s()` updates that list. 
The better name for this function would be: `updateDistributableERC20s()`


[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441)
```solidity
441:     function setDistributableERC20s(
442:         address[] memory _distributableERC20s
443:     ) public onlyOwner {
444:         distributableERC20s = _distributableERC20s;
445:     }
```



# [8] Declaring iterator variables outside the loop decreases the code readability

**File:** `LiquidInfrastructureERC20.sol`

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L213)
```solidity
213:         uint i;
214:         for (i = nextDistributionRecipient; i < limit; i++) {
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L370)
```solidity
370:         uint256 i;
371:         for (i = nextWithdrawal; i < limit; i++) {
```

Across the whole code-base, the iterator is being declared within the loop, e.g.,: `for (uint j = 0; j < distributableERC20s.length; j++)`.
The only exceptions have been noticed at lines 213-214 and lines 370-371. Our recommendation is to stick to one way of loop-syntax and declare iterators within the loop. Declaring iterators inside the loop increases the code readability:

```
for (uint i = nextDistributionRecipient; i < limit; i++) {
```

# [9] Incorrect comment for `mint()`

**File:** `LiquidInfrastructureERC20.sol`

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L313)
```solidity
313:     /**
314:      * Allows the contract owner to mint tokens for an address
315:      *
316:      * @notice minting may only occur when a distribution has happened within MinDistributionPeriod blocks
317:      */
318:     function mint(
319:         address account,
320:         uint256 amount
321:     ) public onlyOwner nonReentrant {
322:         _mint(account, amount);
323:     }
```

According to the comment: `minting may only occur when a distribution has happened within MinDistributionPeriod blocks`. However, according to the code-base - function `mint()` can be called anytime.
Please notice this is a public function, thus `onlyOwner` is able to call it anytime.
Secondly, function calls `_mint()` from ERC-20. The `_mint()` implementation straightforwardly mints tokens, without checking any additional conditions.
Our recommendation is to rewrite this comment, so it will be more clear and easier to understand. 

# [10] Code which repeats may be refactored into modifier

**File:** `LiquidInfrastructureERC20.sol`

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L303)
```solidity
303:     function mintAndDistribute(
304:         address account,
305:         uint256 amount
306:     ) public onlyOwner {
307:         if (_isPastMinDistributionPeriod()) {
308:             distributeToAllHolders();
309:         }
310:         mint(account, amount);
311:     }
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L331)
```solidity
331:     function burnAndDistribute(uint256 amount) public {
332:         if (_isPastMinDistributionPeriod()) {
333:             distributeToAllHolders();
334:         }
335:         burn(amount);
336:     }
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L344)
```solidity
344:     function burnFromAndDistribute(address account, uint256 amount) public {
345:         if (_isPastMinDistributionPeriod()) {
346:             distributeToAllHolders();
347:         }
348:         burnFrom(account, amount);
349:     }
```

The same code-block: 
```
        if (_isPastMinDistributionPeriod()) {
            distributeToAllHolders();
```

can be refactored into a modifier, since it's being called by 3 different functions: `mintAndDistribute()`, `burnAndDistribute()`, `burnFromAndDistribute()`

# [11] Style guide: Public variables should start with lowercase letter

**File:** `LiquidInfrastructureERC20.sol`

Please notice that this issue was missed by the bot. During the bot-race, the Style guide checker detected only:
* [N-61] Style guide: Non-external/public variable names should begin with an underscore
* [N-62] Style guide: Variable names for constants are improperly named



According to the Solidity Style Guide, public variables should start with lowercase letter. E.g., `uint256 public MinDistributionPeriod` should be `uint256 public minDistributionPeriod`

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L60)
```solidity
60:     address[] public ManagedNFTs;
[...]
65:     mapping(address => bool) public HolderAllowlist;
[...]
70:     uint256 public LastDistribution;
[...]
75:     uint256 public MinDistributionPeriod;
[...]
80:     bool public LockedForDistribution;
```

# [12] Redundant `require` statement

**File:** `LiquidInfrastructureERC20.sol`

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431)
```solidity
431:         require(true, "unable to find released NFT in ManagedNFTs");
```

Above `require` statement is redundant - since it will always evaluate to `true`. 

# [13] Typos 

**File:** `LiquidInfrastructureNFT.sol`

[File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L100)
```solidity
100:      * The ERC20s specified here have EVM addressses, querying the x/erc20 module will reveal their
```

`addressses` should be changed to `addresses`.

# [14] Incorrect punctuation

**File:** `OwnableApprovableERC721.sol`

[File: liquid-infrastructure/contracts/OwnableApprovableERC721.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L11)
```solidity
11:  * @dev For contracts with manually enumerated tokens (e.g. RockId = 1, PaperId = 2, ScissorsId = 3) use the modifiers like so:
```

There should be comma after `e.g.`. `(e.g. RockId = 1)` should be changed to `(e.g., RockId = 1)`

# [15] Use `uint256` instead of `uint`

**Files:** `LiquidInfrastructureNFT.sol`, `LiquidInfrastructureERC20.sol`

Even though `uint` is a shorter form of `uint256` - using `uint256` increases the code readability. 
We have detected that `uint256` is used across the whole code-base. The only exceptions were noticed in the below instances:

[File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L171)
```solidity
171:             for (uint i = 0; i < holders.length; i++) {
213:         uint i;
220:                 for (uint j = 0; j < distributableERC20s.length; j++) {
271:         for (uint i = 0; i < distributableERC20s.length; i++) {
421:         for (uint i = 0; i < ManagedNFTs.length; i++) {
467:         for (uint i = 0; i < _approvedHolders.length; i++) {
```

[File: liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L120)
```solidity
120:         for (uint i = 0; i < newErc20s.length; i++) {
175:         for (uint i = 0; i < erc20s.length; i++) {
```
