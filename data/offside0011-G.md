
# [G-01] Use Solidity array assignment instead of manually assign arrays


```c
File: LiquidInfrastructureNFT.sol
108:     function setThresholds(
109:         address[] calldata newErc20s,
110:         uint256[] calldata newAmounts
111:     ) public virtual onlyOwnerOrApproved(AccountId) {
112:         require(
113:             newErc20s.length == newAmounts.length,
114:             "threshold values must have the same length"
115:         );
116:         // Clear the thresholds before overwriting
117:         delete thresholdErc20s;
118:         delete thresholdAmounts;
119: 
120:         for (uint i = 0; i < newErc20s.length; i++) {
121:             thresholdErc20s.push(newErc20s[i]);
122:             thresholdAmounts.push(newAmounts[i]);
123:         }

```

it is equivalent of:

```c
thresholdErc20s = newErc20s;
thresholdAmounts = newAmounts;
```

# [G-02] Use Constants to avoid extra static call


`nft.AccountId()` will compile to an extra static call to get the constant `1`.

```c
File: LiquidInfrastructureERC20.sol
394:     function addManagedNFT(address nftContract) public onlyOwner {
395:         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
396:         address nftOwner = nft.ownerOf(nft.AccountId());


File: LiquidInfrastructureERC20.sol
413:     function releaseManagedNFT(
414:         address nftContract,
415:         address to
416:     ) public onlyOwner nonReentrant {
417:         LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
418:         nft.transferFrom(address(this), to, nft.AccountId());
```

define the constant to save gas:

```solidity
uint256 public constant AccountId = 1;
```

# [G-03] Remove debug require in release code

```c
File: LiquidInfrastructureERC20.sol
430:         // By this point the NFT should have been found and removed from ManagedNFTs
431:         require(true, "unable to find released NFT in ManagedNFTs");
```

# [G-04] Eliminate redundant length check while clearing an array

The `if (erc20EntitlementPerUnit.length > 0)` condition can be safely removed without causing any disruption.

```c
File: LiquidInfrastructureERC20.sol
257:     function _beginDistribution() internal {
258:         require(
259:             !LockedForDistribution,
260:             "cannot begin distribution when already locked"
261:         );
262:         LockedForDistribution = true;
263: 
264:         // clear the previous entitlements, if any
265:         if (erc20EntitlementPerUnit.length > 0) {
266:             delete erc20EntitlementPerUnit;
267:         }

```


# [G-05] Change `!(==)` to `!=`

Replace `if (!(to == address(0)))` with `if (to != address(0))`

```c
File: LiquidInfrastructureERC20.sol
127:     function _beforeTokenTransfer(
128:         address from,
129:         address to,
130:         uint256 amount
131:     ) internal virtual override {
132:         require(!LockedForDistribution, "distribution in progress");
133:         if (!(to == address(0))) {

```

# [G-06] use notExists to avoid negation over negation

change

```c
File: LiquidInfrastructureERC20.sol
142:         bool exists = (this.balanceOf(to) != 0);
143:         if (!exists) {
144:             holders.push(to);
145:         }

```

to 

```c
bool notExists = this.balanceOf(to) == 0;
if (notExists) { ... }
```

and the following code can also be optmized:

```c
File: LiquidInfrastructureERC20.sol
169:         bool stillHolding = (this.balanceOf(from) != 0);
170:         if (!stillHolding) {

```
