0. QA: `LiquidInfrastructureERC20::constructor()` - Although OZ version v4.3.1 should be sufficient for this deployment version, recommended to use the latest version of OZ, v5.0.0, which initializes the contract setting the address provided by the deployer as the initial owner.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L408
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L5

`@openzeppelin/contracts/access/Ownable.sol`:
```solidity
constructor(address initialOwner) {
```
Example implementation:
`LiquidInfrastructureERC20:constructor()`:
```solidity
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s,
+       address _initialOwner
    ) ERC20(_name, _symbol) Ownable(_initialOwner) {
```

=======
=======

1. QA: Dev comment may be somewhat confusing/unclear. 

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L27

Dev comment may be somewhat confusing/unclear:
`"Minting and burning of this ERC20 is restricted if the minimum distribution period has elapsed, and it is reenabled once a new distribution is complete."`

The above dev comment implies that minting/burning is allowed during the minimum distribution period. But then the confusing comment of `"it is reenabled once a new distribution is complete."`

Some questions:
- Maybe the distribution continues after minimum distribution period?
- And that the minting/burning is only allowed during minimum distribution period?

=======
=======

2. QA: Missing `indexed` in event declarations.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L30-L38
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L35-L38

Taking as examples:
```solidity
event SuccessfulWithdrawal(address to, address[] erc20s, uint256[] amounts);
```
```solidity
event Distribution(address recipient, address[] tokens, uint256[] amounts);
```
The `indexed` keyword in Solidity event declarations allows for efficient filtering and searching of events, leading to lower gas costs and improved search efficiency, especially for frequently occurring events or when optimizing gas usage. Marking parameters like `to` as `indexed` in events such as `SuccessfulWithdrawal` can improve search efficiency for targeted log retrieval based on specific criteria like the `to` address.

=======
=======

3. QA/LOW: `LiquidInfrastructureERC20::MinDistributionPeriod()` - the code implementation at L220 does not reflect the natspec comment for this state variable.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L69

Take note of the `"required to elapse before"` in below natspec comment, specifically the `"elapse"` word. It clearly means AFTER. I.e. new distribution can only begin AFTER the `MinDistributionPeriod` has ELAPSED, but the code implementation allows for new distribution period to begin AT/during the last block of the `MinDistributionPeriod`, which is an incorrect implementation. This implementation bug is covered in a different bug report. This QA/LOW report is regarding the inconsistency between the state variable declaration's natspec comment and how it is implemented/used in the codebase.

```solidity
    /**
     * @notice Holds the minimum number of blocks required to elapse before a new distribution can begin
     */
    uint256 public MinDistributionPeriod;
```
And the implementation at L220:
```solidity
return (block.number - LastDistribution) >= MinDistributionPeriod;
```
I cover this incorrect implementation in a separate bug report, either medium or high.

=======
=======

4. QA: Several instances where `public` modifier is used with no internal calls to the function, therefore the `external` visibility modifier should be used, in line with standard recommendations & best practices.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L100
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L110
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L267
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L289
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L302
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L309
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L347
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L363
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L388
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L85
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L109
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L136
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L204

The affected functions are:
`approveHolder()`,
`disapproveHolder()`,
`mintAndDistribute()`,
`burnAndDistribute()`,
`burnFromAndDistribute()`,
`withdrawFromAllManagedNFTs()`,
`addManagedNFT()`,
`releaseManagedNFT()`,
`setDistributableERC20s()`,
`getThresholds()`,
`setThresholds()`,
`withdrawBalances()`,
`recoverAccount()`

=======
=======

5. LOW: `LiquidInfrastructureERC20::approveHolder()` - Owner can make himself or this contract a holder.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L94-L103

Although owner can easily just add any one of his own addresses thereby becoming a holder, at the very least he should not be allowed to make his own `msg.sender` address or `address(this)` a holder, to help maintain trust in the protocol.
```solidity
    function approveHolder(address holder) public onlyOwner {
        require(!isApprovedHolder(holder), "holder already approved");
        HolderAllowlist[holder] = true;
    }
```

=======
=======

6. QA: Three internal functions not currently used: `LiquidInfrastructureERC20::_beforeTokenTransfer()`, `LiquidInfrastructureERC20::_beforeMintOrBurn()` and `LiquidInfrastructureERC20::_afterTokenTransfer()`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L115-L159

Consider commenting out or removing these three unused functions unless there's a plan to use them in future upgrades. They will add to deployment gas cost overhead in a contract that already contains functions that are gas guzzlers.👀

=======
=======

7. LOW: Approved `holder` address can be added to the `holders` array more than once, as long as the holder's balance is zeroed out/emptied before calling `_beforeTokenTransfer()`. Can effectively receive 2x or more rewards.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L131

LOW severity because the affected internal functions are NOT in use, otherwise could have been medium or high severity depending.

=======
=======

8. QA: Dev should add comments/natspec to make it clear that the unused internal functions `_beforeTokenTransfer()` & `_afterTokenTransfer()` are replaced by `_beginDistribution()` & `_endDistribution()` respectively?

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L121
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L148
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L229
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L253

=======
=======

9. QA: `LiquidInfrastructureERC20::distributeToAllHolders()` - Should handle the case where `holders.length = 0`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L166

Recommendation:
```diff
    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
            distribute(holders.length);
+       } else {
+       	revert HoldersArrayEmpty();
+       }
    }
```
Or if dont want to revert, add a suitable event instead.

=======
=======

10. QA: no reentrancy protection for `withdrawFromManagedNFTs()`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L317

```solidity
function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
```
Impact:
- brief checking doesnt seem any high risks

Recommendation:
- consider adding reentrancy protection

=======
=======

11. QA: Function `releaseManagedNFT()` should use `safeTransferFrom()` instead.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L365

```solidity
nft.transferFrom(address(this), to, nft.AccountId());
```

Recommendation:

Use OZ's `safeTransferFrom()`.

=======
=======

12. QA: `LiquidInfrastructureERC20::constructor()` - Since `MinDistributionPeriod` cannot be updated after deployment, unless upgrade, should have check to ensure value != 0.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L416

```diff
+	require(_minDistributionPeriod != 0, "_minDistributionPeriod cannot be zero");
	MinDistributionPeriod = _minDistributionPeriod;
```
=======
=======

13. QA: `LiquidInfrastructureNFT::constructor()` - Unless there's a valid reason, should definitely use `_safeMint` instead.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L74

"Usage of this method is discouraged, use {_safeMint} whenever possible".

```solidity
_mint(msg.sender, AccountId);
```
