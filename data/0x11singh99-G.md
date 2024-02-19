## Gas Optimizations

**Note : G-02 and G-04 contains only those instances which were missed by bot.**

| Issue                                                                                                                                                                                                                                                                                                                                            |   Total Gas Saved    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------: |
| [[G-01] **Cache `erc20EntitlementPerUnit` and `distributableERC20s` storage arrays in memory outside of nested for loops to avoid multiple `Gwarmsload` for every `i` value.**](#g-01-cache-erc20entitlementperunit-and-distributableerc20s-storage-arrays-in-memory-outside-of-nested-for-loops-to-avoid-multiple-gwarmsload-for-every-i-value) | Mentioned in finding |
| [[G-02] **State variables can be packed into fewer storage slot (saves ~2000 Gas)(instance Missed by bot)**](#g-02-state-variables-can-be-packed-into-fewer-storage-slot-saves-2000-gasinstance-missed-by-bot)                                                                                                                                   |        ~2000         |
| [[G-03] **Unnecessary `nonReentrant` modifier checks in functions(saves ~24000 GAS)**](#g-03-unnecessary-nonreentrant-modifier-checks-in-functionssaves-24000-gas)                                                                                                                                                                               |        ~24000        |
| [[G-04] **Use already `cached` value instead of re-reading from `storage`( instance Missed by bot)**](#g-04-use-already-cached-value-instead-of-re-reading-from-storage-instance-missed-by-bot)                                                                                                                                                  |         ~100         |
| [[G-05] **No need to `inherit` `ERC721` again**](#g-05-no-need-to-inherit-erc721-again)                                                                                                                                                                                                                                                          |          -           |
| [[G-06] **`Remove` unnecessary `require` statements**](#g-06-remove-unnecessary-require-statements)                                                                                                                                                                                                                                              |         ~115         |

## [G-01] Cache `erc20EntitlementPerUnit` and `distributableERC20s` storage arrays in memory outside of nested for loops to avoid multiple `Gwarmsload` for every `i` value.

Since two `for` loops are nested here. So internal for loop will run `distributableERC20s.length` times for every `i` value. That means every element in `distributableERC20s` and `distributableERC20s.length` elements in `erc20EntitlementPerUnit` array will be accessed for `limit - nextDistributionRecipient` times ie. for each `i` value.

By caching those arrays outside of both loops can result in accessing those array's `distributableERC20s.length` no. of starting elements only one times instead of multiple times.

It can save total **2 \* `limit - nextDistributionRecipient - 1` \* `distributableERC20s.length`** `SLOAD`. Which can save ~100 Gas (( 97 Gas technically)) on avoiding each SLOAD when accessed after first time . Since each `SLOAD` cost 100 Gas while each MLOAD takes only 3 Gas.

### Total Gas Saved: **2 \* `limit - nextDistributionRecipient - 1` \* `distributableERC20s.length` \* 97** Gas will be saved.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

214:        for (i = nextDistributionRecipient; i < limit; i++) {
215:            address recipient = holders[i];
216:            if (isApprovedHolder(recipient)) {
217:                uint256[] memory receipts = new uint256[](
218:                    distributableERC20s.length
219:                );
220:                for (uint j = 0; j < distributableERC20s.length; j++) {
221:                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                      //@audit cache `erc20EntitlementPerUnit` and `distributableERC20s` outside from here and access here from memory
222:                    uint256 entitlement = erc20EntitlementPerUnit[j] *
223:                        this.balanceOf(recipient);
224:                    if (toDistribute.transfer(recipient, entitlement)) {
225:                        receipts[j] = entitlement;
226:                    }
227:                }
228:
229:                emit Distribution(recipient, distributableERC20s, receipts);
230:            }
231:        }

```

**Recommended Mitigation Steps:**

```diff
+           uint256[] memory m_erc20EntitlementPerUnit = new uint256[](distributableERC20s.length);
+           address[] memory m_distributableERC20s = new address[](distributableERC20s.length);
+           m_erc20EntitlementPerUnit = erc20EntitlementPerUnit;
+           m_distributableERC20s = distributableERC20s;

214:        for (i = nextDistributionRecipient; i < limit; i++) {
215:            address recipient = holders[i];
216:            if (isApprovedHolder(recipient)) {
217:                uint256[] memory receipts = new uint256[](
218:                    distributableERC20s.length
219:                );
220:                for (uint j = 0; j < distributableERC20s.length; j++) {
- 221:                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
- 222:                    uint256 entitlement = erc20EntitlementPerUnit[j] *
+ 221:                    IERC20 toDistribute = IERC20(m_distributableERC20s[j]);
+ 222:                    uint256 entitlement = m_erc20EntitlementPerUnit[j] *
223:                        this.balanceOf(recipient);
224:                    if (toDistribute.transfer(recipient, entitlement)) {
225:                        receipts[j] = entitlement;
226:                    }
227:                }
228:
229:                emit Distribution(recipient, distributableERC20s, receipts);
230:            }
231:        }

```

[LiquidInfrastructureERC20.sol#L214C1-L231C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214C1-L231C10)

## [G-02] State variables can be packed into fewer storage slot (saves 2000 Gas)(instance Missed by bot)

### Total Gas Saved: `2000 GAS, 1 SLOT`

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper when accessed in same transaction.

### `LastDistribution` and `LockedForDistribution` can be packed in `single` slot `SAVES: 2000 GAS, 1 Storage SLOTs`

**`LastDistribution`** This can be reduced to `uint48` since it holds block number so uint48 is more than sufficient to hold any realistic block number of any blockchain.

**Note: It only include instance which was missed by bot. `MinDistributionPeriod` already covered in bot report. So it doesn't included here. While `LastDistribution` reducing size is not covered in bot report. And it is major gas saving so it is covered here.**

```solidity
File : contracts/LiquidInfrastructureERC20.sol

70:    uint256 public LastDistribution;

75:    uint256 public MinDistributionPeriod;

80:    bool public LockedForDistribution;

```

[LiquidInfrastructureERC20.sol#L70-L80](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L70-L80)

**Recommended Mitigation Steps:**

```diff
File : contracts/LiquidInfrastructureERC20.sol

-70:    uint256 public LastDistribution;


75:     uint256 public MinDistributionPeriod;

+       uint48 public LastDistribution;
80:     bool public LockedForDistribution;

```

## [G-03] Unnecessary `nonReentrant` modifier checks in functions(saves ~24000 GAS)

### `SAVES ~24000 GAS` when every time respective function called

Remove the unnecessary `nonReentrant` modifier since this function does not contain any external calls, and it is not invoked within the `_mint` function flow. This function `mint` not called again in `_mint` flow also not in overridden `_beforeTokenTransfer` and `_afterTokenTransfer`.So there is no chance of reentrancy in this context. The `_mint` function, which is part of the OpenZeppelin library, includes hooks before and after token transfer, and they do not involve any external calls in their overridden functions.and `_mint` already doesn't have any external call. Removing the `nonReentrant` modifier results in a gas saving of approximately _~24700_ gas.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

318: function mint(
319:    address account,
320:    uint256 amount
321:   ) public onlyOwner nonReentrant {
322:     _mint(account, amount);
323:    }

```

[LiquidInfrastructureERC20.sol#L318C4-L323C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L318C4-L323C6)

**Recommended Mitigation Steps:**

```diff
File : contracts/LiquidInfrastructureERC20.sol

318: function mint(
319:    address account,
320:    uint256 amount
-321:   ) public onlyOwner nonReentrant {
+321:   ) public onlyOwner  {
322:     _mint(account, amount);
323:    }

```

## [G-04] Use already `cached` value instead of re-reading from `storage`( instance Missed by bot)

Since reading state var. from storage takes 100 Gas in other consecutive after first access. So to avoid 1 SLOAD we can use already cached value.

**Note: It only include those instances which were missed by bot**

### Total Gas Saved : ~100 Gas

```solidity
File : liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

184:    function distributeToAllHolders() public {
185:          uint256 num = holders.length;
186:          if (num > 0) {
187:              distribute(holders.length);
188:          }
189:      }

```

[LiquidInfrastructureERC20.sol#L184C4-L189C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184C4-L189C6)

**Recommended Mitigation Steps:**

```diff
File : liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
-            distribute(holders.length);
+            distribute(num);
        }
    }

```

## [G-05] No need to `inherit` `ERC721` again

Since `OwnableApprovableERC721` already inherits `ERC721` so no need to inherit `ERC721` again in `LiquidInfrastructureNFT` contract.

```solidity
20: abstract contract OwnableApprovableERC721 is Context, ERC721 {
```

There is no need to explicitly inherit `ERC721` again in this context. Additionally removing the unnecessary import of `ERC721` above will streamline the code.

```solidity
File : contracts/LiquidInfrastructureNFT.sol

5:  import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

...

33: contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {

```

[LiquidInfrastructureNFT.sol#L5C1-L33C70](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L5C1-L33C70) ,
[OwnableApprovableERC721.sol#L20](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L20)

**Recommended Mitigation steps:**

```diff
File : contracts/LiquidInfrastructureNFT.sol

-5:  import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

...

-33: contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
+33: contract LiquidInfrastructureNFT is  OwnableApprovableERC721 {

```

## [G-06] `Remove` unnecessary `require` statements

### Instance#1

**Saves ~115 Gas on every time `_beginDistribution` function called.**

Since `_beginDistribution` function called only once inside `distribute` when `LockedForDistribution` is false.
So `LockedForDistribution` will always be false when `_beginDistribution` will be called. So there is no requirement of `require` check in `_beginDistribution` starting at line 258.
It can save 1 SLOAD (~100 Gas) + 1 NOT opcode(~3 gas) + 1 require check which will always be true(~15 Gas).

```solidity
File : contracts/LiquidInfrastructureERC20.sol

      function distribute(uint256 numDistributions) public nonReentrant {
200:        if (!LockedForDistribution) {
201:            require(
202:                _isPastMinDistributionPeriod(),
203:                "MinDistributionPeriod not met"
204:            );
205:            _beginDistribution();
206:        }
```

**Remove the below require statement**

```solidity
File : contracts/LiquidInfrastructureERC20.sol

257:    function _beginDistribution() internal {
258:        require(
259:            !LockedForDistribution,
260:            "cannot begin distribution when already locked"
263:        );//@audit gas remove this unnecessary require
264:        LockedForDistribution = true;

```

[LiquidInfrastructureERC20.sol#L200C1-L206C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L200C1-L206C10) ,
[LiquidInfrastructureERC20.sol#L257C5-L262C38](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L257C5-L262C38)

### Instance#2

`require` will never revert here so remove it. It doesn't have any utility here only wasting little gas in executing and always pass.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

431: require(true, "unable to find released NFT in ManagedNFTs");

```

[LiquidInfrastructureERC20.sol#L431](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431)

```diff
File : contracts/LiquidInfrastructureERC20.sol

- 431: require(true, "unable to find released NFT in ManagedNFTs");

```
