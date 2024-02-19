## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls	 | 3 | - |
| [G-02] | Using fixed bytes is cheaper than using string | 2 | - |
| [G-03] | Use  `bytes.concat()` instead of `string.concat()` | 1 | - |
| [G-04] | Using `calldata` instead of `memory` for read-only arguments in external / public functions saves gas | 1 | - |
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | 1 | - |
| [G-06] | Use bitmap to save gas | 4 | - |
| [G-07] | State Variable can be packed into fewer storage slots | 1 | - |
| [G-08] | Using storage instead of memory for structs/arrays saves gas | 4 | - |
| [G-09] | Use assembly to write address storage values | 2 | - |
| [G-10] | Use hardcode address instead address(this) | 4 | - |
| [G-11] | Use uint256(1)/uint256(2) instead for `true` and `false` boolean states | 3 | - |
| [G-12] | Use assembly to validate `msg.sender` | 1 | - |
| [G-13] | Pre-increment and pre-decrement are cheaper than +1 ,-1 | 2 | - |
| [G-14] | Use assembly for loops | 1 | - |
| [G-15] | Cache external calls outside of loop to avoid re-calling function on each iteration | 1 | - |
| [G-16] | Simple checks for zero uint can be done using assembly to save gas | 1 | - |


## Gas Optimizations  

## [G-01] Avoid contract existence checks by using low level calls		


```solidity
file: /LiquidInfrastructureERC20.sol

396        address nftOwner = nft.ownerOf(nft.AccountId());

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L396C1-L396C57


```solidity
file: LiquidInfrastructureERC20.sol

//@audit getThresholds()  , withdrawBalance  

376            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
377            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376C1-L377C76


## [G-02]  Using fixed bytes is cheaper than using string



```solidity
file: /LiquidInfrastructureERC20.sol

457        string memory _name,
458        string memory _symbol,

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L457C1-L458C31


## [G-03] Use   `bytes.concat()` instead of `string.concat()`
`bytes` operations are more gas-efficient than equivalent `string` operations because Solidity's bytes type is more low-level and does not have the same level of abstraction as string.`


```solidity
file: /LiquidInfrastructureNFT.sol

65        ERC721(
            string.concat(
                "althea://liquid-infrastructure-account/",
                accountName
            ),
            string.concat("LIA:", accountName)
        )

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L65C1-L71C10

### Recommended code:

```solidity
ERC721(
    bytes.concat(
        "althea://liquid-infrastructure-account/",
        accountName
    ),
    bytes.concat("LIA:", accountName)
)
```

## [G-04] Using `calldata` instead of `memory` for read-only arguments in external / public functions saves gas 


```solidity
file: /LiquidInfrastructureERC20.sol

441    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441C1-L443C25


## [G-05] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything


```solidity
file: /LiquidInfrastructureERC20.sol

418        nft.transferFrom(address(this), to, nft.AccountId());

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L418C1-L418C62


## [G-06] Use bitmap to save gas

```solidity
file: /LiquidInfrastructureERC20.sol

65    mapping(address => bool) public HolderAllowlist;

108        HolderAllowlist[holder] = true;

118        HolderAllowlist[holder] = false;

262        LockedForDistribution = true;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L108C1-L108C40


## [G-07] State Variable can be packed into fewer storage slots   

```solidity
file: /LiquidInfrastructureERC20.sol

/// @audit if the  'address' and 'bool' types variable comes one after another, than this will be reserve one slot. 


54    uint256 public constant Version = 1;

    /**
     * @notice This collection holds the managed LiquidInfrastructureNFTs which periodically generate revenue and deliver
     * the balances to this contract.
     */
60    address[] public ManagedNFTs;

    /**
     * @notice This collection holds the whitelist for accounts approved to hold the LiquidInfrastructureERC20
     */
65    mapping(address => bool) public HolderAllowlist;

    /**
     * @notice Holds the block of the last distribution, used for limiting distribution lock ups
     */
70    uint256 public LastDistribution;

    /**
     * @notice Holds the minimum number of blocks required to elapse before a new distribution can begin
     */
75    uint256 public MinDistributionPeriod;

    /**
     * @notice When true, locks all transfers, mints, and burns until the current distribution has completed
     */
80    bool public LockedForDistribution;


```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L54C2-L80C39


## [G-08] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /LiquidInfrastructureERC20.sol

376            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();

442        address[] memory _distributableERC20s

459        address[] memory _managedNFTs,
460        address[] memory _approvedHolders,

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376C1-L376C80


## [G-09] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```solidity
file: /LiquidInfrastructureERC20.sol

///@audit 'distributableERC20s' and 'ManagedNFTs' are state adress type variabls.

444        distributableERC20s = _distributableERC20s;

401        ManagedNFTs.push(nftContract);


```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L444


## [G-10] Use hardcode address instead address(this)

```solidity
file: /LiquidInfrastructureERC20.sol

272           uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );

377            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));

418        nft.transferFrom(address(this), to, nft.AccountId());

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L272C5-L274C15


```solidity
file: LiquidInfrastructureNFT.sol

177            uint256 balance = IERC20(erc20).balanceOf(address(this));

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L177C1-L177C70


## [G-11] Use uint256(1)/uint256(2) instead for `true` and `false` boolean states


If you don't use boolean for storage you will avoid Gwarmaccess 100 gas. In addition, state changes of boolean from `true` to `false` can cost up to ~20000 gas rather than `uint256(2)` to `uint256(1)` that would cost significantly less.

```solidity
file: /contracts/LiquidInfrastructureERC20.sol

65    mapping(address => bool) public HolderAllowlist;

108        HolderAllowlist[holder] = true;

118        HolderAllowlist[holder] = false;

468            HolderAllowlist[_approvedHolders[i]] = true;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L65C1-L65C53


## [G-12] Use assembly to validate `msg.sender`

```solidity
file: /contracts/LiquidInfrastructureNFT.sol
 
73        _mint(msg.sender, AccountId);

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L73C1-L73C38


## [G-13] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
file: /LiquidInfrastructureERC20.sol

174                    holders[i] = holders[holders.length - 1];

425                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L174C1-L174C62


## [G-14] Use assembly for loops

In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

```solidity
file: /contracts/LiquidInfrastructureERC20.sol

171            for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L171C1-L177C14


## [G-15] Cache external calls outside of loop to avoid re-calling function on each iteration

```solidity
file: /LiquidInfrastructureERC20.sol

376            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376C1-L376C80


## [G-16] Simple checks for zero uint can be done using assembly to save gas

```solidity
file: /LiquidInfrastructureERC20.sol

362        if (nextWithdrawal == 0) {

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L362C1-L362C35
