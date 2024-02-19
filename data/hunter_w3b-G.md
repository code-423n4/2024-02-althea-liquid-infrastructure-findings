# Gas-Optimization For `Altea Liquid Infrastructure`

## [G-01] A `MODIFIER` used only once and not being inherited should be inlined to save gas

These modifiers that are utilized only once and are being inherited. This suggests an opportunity for gas optimization through inlining these modifiers to reduce gas during contract deployment and execution.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L24

```solidity
File: contracts/OwnableApprovableERC721.sol

24    modifier onlyOwner(uint256 tokenId) {
25        require(
26            ERC721(this).ownerOf(tokenId) == _msgSender(),
27            "OwnableApprovable: caller is not the owner"
28        );
29        _;
30    }


35     modifier onlyOwnerOrApproved(uint256 tokenId) {
36        // Get approval directly from ERC721's internal method
37        if (_isApprovedOrOwner(_msgSender(), tokenId)) {
38            _;
39        } else {
40            revert("OwnableApprovable: caller is not owner nor approved");
41        }
```

**They used only once in this contract**

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108

```solidity
File: contracts/LiquidInfrastructureNFT.sol

108    function setThresholds(
109        address[] calldata newErc20s,
110        uint256[] calldata newAmounts
111    ) public virtual onlyOwnerOrApproved(AccountId) {



203    function recoverAccount() public virtual onlyOwner(AccountId) {
204        emit TryRecover();
205    }
```

## [G-02] State variables should be cached in stack variables rather than re-reading them from storage

### There are missed from bots

**gas saving: `32 * 97` = 3104**

```solidity
File: contracts/LiquidInfrastructureERC20.sol

/// @audit erc20EntitlementPerUnit  is State Variable
265        if (erc20EntitlementPerUnit.length > 0) {
266            delete erc20EntitlementPerUnit;
276            erc20EntitlementPerUnit.push(entitlement);


/// @audit holders  is State Variable
171            for (uint i = 0; i < holders.length; i++) {
172                if (holders[i] == from) {
173                    // Remove the element at i by copying the last one into its place and removing the last element
174                    holders[i] = holders[holders.length - 1];
175                    holders.pop();


/// @audit nextDistributionRecipient  is State Variable
209            nextDistributionRecipient + numDistributions,
214        for (i = nextDistributionRecipient; i < limit; i++) {
232        nextDistributionRecipient = i;
234        if (nextDistributionRecipient == holders.length) {


/// @audit LockedForDistribution  is State Variable
259             !LockedForDistribution,
262        LockedForDistribution = true;


/// @audit LockedForDistribution  is State Variable
288            LockedForDistribution,
292        LockedForDistribution = false;



/// @audit nextWithdrawal  is State Variable
362        if (nextWithdrawal == 0) {
367            numWithdrawals + nextWithdrawal,
371        for (i = nextWithdrawal; i < limit; i++) {
380        nextWithdrawal = i;
382        if (nextWithdrawal == ManagedNFTs.length) {
383            nextWithdrawal = 0;


/// @audit ManagedNFTs  is State Variable
421        for (uint i = 0; i < ManagedNFTs.length; i++) {
422            address managed = ManagedNFTs[i];
423            if (managed == nftContract) {
424                // Delete by copying in the last element and then pop the end
425                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
426                ManagedNFTs.pop();
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L265

```solidity
File: contracts/LiquidInfrastructureNFT.sol

117        delete thresholdErc20s;
121            thresholdErc20s.push(newErc20s[i]);

118        delete thresholdAmounts;
122            thresholdAmounts.push(newAmounts[i]);


137            _isApprovedOrOwner(_msgSender(), AccountId),
140        address destination = ownerOf(AccountId);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L117

## [G-03] Refactor internal function to avoid unnecessary SLOAD

```solidity
File: contracts/LiquidInfrastructureERC20.sol

198    function distribute(uint256 numDistributions) public nonReentrant {
199        require(numDistributions > 0, "must process at least 1 distribution");
200        if (!LockedForDistribution) {
201            require(
202                _isPastMinDistributionPeriod(),
203                "MinDistributionPeriod not met"
204            );
205            _beginDistribution();  // @audit  we can  cache the LockedForDistribution when we call the internal function
206        }



257    function _beginDistribution() internal {
258        require(
259            !LockedForDistribution, // @audit  we can  cache the LockedForDistribution in distribute function to avoid uncecessary SLOAD
260            "cannot begin distribution when already locked"
261        );
262        LockedForDistribution = true;
263
264        // clear the previous entitlements, if any
265        if (erc20EntitlementPerUnit.length > 0) {
266            delete erc20EntitlementPerUnit;
267        }



286    function _endDistribution() internal {
287        require(
288            LockedForDistribution,// @audit  we can  cache the LockedForDistribution in distribute function to avoid uncecessary
289            "cannot end distribution when not locked"
290        );
291        delete erc20EntitlementPerUnit;
292        LockedForDistribution = false;
293        LastDistribution = block.number;
294        emit DistributionFinished();
295    }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198

## [G-04] Avoid Making any STATE reads if we can revert early (Save 2100 Gas)

Avoid reading from storage if we might revert early. The first check involves making a state read, `LockedForDistribution` The second check, the if statement, checks if a function parameter `to` is equal to `address(0)` and reverts if it happens to be zero. As it is, if `to` is zero address, we would consume gas in the require statement to make the state read(`2100 Gas for the state read`) and then revert on the second check we can save some gas in the case of such a revert by moving the cheap check to the top and only reading state variable after validating the function parameter.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

127    function _beforeTokenTransfer(
128        address from,
129        address to,
130        uint256 amount
131    ) internal virtual override {
132        require(!LockedForDistribution, "distribution in progress");
133        if (!(to == address(0))) {
134            require(
135                isApprovedHolder(to),
136                "receiver not approved to hold the token"
137            );
138        }
139        if (from == address(0) || to == address(0)) {
140            _beforeMintOrBurn();
141        }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L127-L140

## [G-05] Mappings used within a function more than once should be cached to save gas

### these are missed from the bot

```solidity
File: contracts/LiquidInfrastructureERC20.sol

218                    distributableERC20s.length
220                for (uint j = 0; j < distributableERC20s.length; j++) {
221                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
229                emit Distribution(recipient, distributableERC20s, receipts);

265        if (erc20EntitlementPerUnit.length > 0) {
266            delete erc20EntitlementPerUnit;

271        for (uint i = 0; i < distributableERC20s.length; i++) {
272            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L218

## [G-06] Use ASSEMBLY to perform efficient back-to-back calls

Using interfaces to make external contract calls in Solidity is convenient but can be inefficient in terms of memory utilization. Each such call involves creating a new memory location to store the data being passed, thus incurring memory expansion costs.

Inline assembly allows for optimized memory usage by re-using already allocated memory spaces or using the scratch space for smaller datasets. This can result in notable gas savings, especially for contracts that make frequent external calls.
Additionally, using inline assembly enables important safety checks like verifying if the target address has code deployed to it using extcodesize(addr) before making the call, mitigating risks associated with contract interactions.

```solidity
File: LiquidInfrastructureNFT.sol

177            uint256 balance = IERC20(erc20).balanceOf(address(this));
179                bool result = IERC20(erc20).transfer(destination, balance);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L177

## [G-07] Shorten the array rather than copying to a new one

ASSEMBLY can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: contracts/LiquidInfrastructureERC20.sol

217                uint256[] memory receipts = new uint256[](
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L217

```solidity
File: contracts/LiquidInfrastructureNFT.sol

174        uint256[] memory amounts = new uint256[](erc20s.length);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L174

## [G-08] Simple checks for zero uint can be done using ASSEMBLY to save gas

```solidity
File: contracts/LiquidInfrastructureERC20.sol

245        if (totalSupply() == 0 || holders.length == 0) {

362        if (nextWithdrawal == 0) {
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L245

## [G-09] Expensive operation inside a for loop

```solidity
File: LiquidInfrastructureERC20.sol

214        for (i = nextDistributionRecipient; i < limit; i++) {
215            address recipient = holders[i];
216            if (isApprovedHolder(recipient)) {
217                uint256[] memory receipts = new uint256[](
218                    distributableERC20s.length
219                );
220                for (uint j = 0; j < distributableERC20s.length; j++) {
221                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
222                    uint256 entitlement = erc20EntitlementPerUnit[j] *
223                       this.balanceOf(recipient);
224                    if (toDistribute.transfer(recipient, entitlement)) {
225                        receipts[j] = entitlement;
226                    }
227                }
228
229                emit Distribution(recipient, distributableERC20s, receipts);
230            }
231        }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214-L231

## [G-10] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

220                for (uint j = 0; j < distributableERC20s.length; j++) {
221                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
222                    uint256 entitlement = erc20EntitlementPerUnit[j] *
223                       this.balanceOf(recipient);
224                    if (toDistribute.transfer(recipient, entitlement)) {
225                        receipts[j] = entitlement;
226                    }
227                }


271        for (uint i = 0; i < distributableERC20s.length; i++) {
272            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
273                address(this)
274            );
275            uint256 entitlement = balance / supply;
276            erc20EntitlementPerUnit.push(entitlement);
277        }


371        for (i = nextWithdrawal; i < limit; i++) {
372            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
373                ManagedNFTs[i]
374            );
375
376            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
377            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
378            emit Withdrawal(address(withdrawFrom));
379        }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L220

```solidity
File: LiquidInfrastructureNFT.sol

175        for (uint i = 0; i < erc20s.length; i++) {
176            address erc20 = erc20s[i];
177            uint256 balance = IERC20(erc20).balanceOf(address(this));
178            if (balance > 0) {
179                bool result = IERC20(erc20).transfer(destination, balance);
180                require(result, "unsuccessful withdrawal");
181               amounts[i] = balance;
182           }
183       }
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L175

## [G-11] Refactor Event to avoid emitting empty data

```solidity
File: contracts/LiquidInfrastructureERC20.sol

280        emit DistributionStarted();

294        emit DistributionFinished();

363            emit WithdrawalStarted();

384            emit WithdrawalFinished();

475        emit Deployed();
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L280

```solidity
File: contracts/LiquidInfrastructureNFT.sol

204        emit TryRecover();

```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L204

## [G-12] Make 3 Event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

36    event Deployed();
37    event DistributionStarted();
38    event Distribution(address recipient, address[] tokens, uint256[] amounts);
39    event DistributionFinished();
40    event WithdrawalStarted();
41    event Withdrawal(address source);
42    event WithdrawalFinished();
43    event AddManagedNFT(address nft);
44    event ReleaseManagedNFT(address nft, address to);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L36

```solidity
File: contracts/LiquidInfrastructureNFT.sol

34    event SuccessfulWithdrawal(address to, address[] erc20s, uint256[] amounts);
35    event TryRecover();
36    event SuccessfulRecovery(address[] erc20s, uint256[] amounts);
37    event ThresholdsChanged(address[] newErc20s, uint256[] newAmounts);
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L34

## [G-13] onlyowner no uses of the nonReentrant modifier

`mint` and `releaseManagedNFT` is only called by the owner if the owner is trusted there is no need for `nonReentrant` save alot of gas.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

321    ) public onlyOwner nonReentrant {

416    ) public onlyOwner nonReentrant {
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L321

## [G-14] Avoid unnecessary public variables

Public storage variables increase the contract's size due to the implicit generation of public getter functions. This makes the contract larger and could increase deployment and interaction costs.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

60    address[] public ManagedNFTs;

70    uint256 public LastDistribution;

75    uint256 public MinDistributionPeriod;

80    bool public LockedForDistribution;
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L60

## [G-15] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

```solidity
File: contracts/LiquidInfrastructureERC20.sol

107        require(!isApprovedHolder(holder), "holder already approved");

117        require(isApprovedHolder(holder), "holder not approved");

132        require(!LockedForDistribution, "distribution in progress");

134            require(

152        require(

199        require(numDistributions > 0, "must process at least 1 distribution");

201            require(

258        require(

287        require(

360        require(!LockedForDistribution, "cannot withdraw during distribution");

397        require(

431        require(true, "unable to find released NFT in ManagedNFTs");
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L107

```solidity
File: contracts/LiquidInfrastructureNFT.sol

112        require(

136        require(

157        require(

180                require(result, "unsuccessful withdrawal");
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L180

## [G-16] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: contracts/LiquidInfrastructureERC20.sol

174                    holders[i] = holders[holders.length - 1];

425                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L425
