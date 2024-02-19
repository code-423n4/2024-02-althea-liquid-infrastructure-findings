
# Gas  optimization 
| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Store Data in calldata Instead of Memory for Certain Function Parameters  | 6 |
|[G-02]| Enable the ABI Encoder v2| 3 |
|[G-03]| Using storage instead of memory for structs/arrays saves gas | 2 |
|[G-04]| Move static calls like address(this) out of loops.| 3 |
|[G-05]| Using assembly to revert with an error message | 17 |
|[G-06]| Don’t make variables public unless necessary | 3 |
|[G-07]| Free Up Unused Storage | 4 |
|[G-08]| Shorten arrays with inline assembly | 2 |
|[G-09]| Using bytes32 is cheaper than using string. Storing and manipulating data in bytes32 is more | 2 |
|[G-10]| Mappings not used externally/internally can be marked private | 1 |


## [G-01]  Store Data in calldata Instead of Memory for Certain Function Parameters 

Instead of copying variables to memory, it is typically more cost-effective to load them immediately from calldata. If all you need to do is read data, you can conserve gas by saving the data in calldata.
Because the values in calldata cannot be changed while the function is being executed, if the variable needs to be updated when calling a function, use memory instead.

```solidity
file: LiquidInfrastructureERC20.sol

442  address[] memory _distributableERC20s

457  string memory _name,

458  string memory _symbol,

459  address[] memory _managedNFTs,

460  address[] memory _approvedHolders,

462  address[] memory _distributableErc20s

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L442

## [G-02] Enable the ABI Encoder v2

The ABI encoder translates data types into the format stored on the blockchain. Enabling the newer v2 encoder saves gas:

```solidity
pragma solidity ^0.8.0;

// Enable ABI encoder v2
pragma abicoder v2;

contract MyContract {
  //...
}
```
. Newer encoder uses less gas for structs, arrays, function params.
. But may break older clients – test compatibility.

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

1  //SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.12; // Force solidity compliance

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L1-L2

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

1  //SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.12; // Force solidity compliance

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L1-L2

```solidity
file: blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol

1 //SPDX-License-Identifier: Apache-2.0
pragma solidity 0.8.12; // Force solidity compliance

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L1-L2

## [G-03] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

217   uint256[] memory receipts = new uint256[](

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L217

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
 
174  uint256[] memory amounts = new uint256[](erc20s.length);

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L174

## [G-04] Move static calls like address(this) out of loops.

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

271  for (uint i = 0; i < distributableERC20s.length; i++) {
            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );
            uint256 entitlement = balance / supply;
            erc20EntitlementPerUnit.push(entitlement);
        }

371   for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L271-L274 

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

175   for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L175-L183

## [G-05] Using assembly to revert with an error message

Issue Description - When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Estimated Gas Savings - Here’s an example:

```solidity
/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}
/// calling restrictedAction(2) with a non-owner address: 23734
contract AssemblyRevert {
    address owner;
    uint256 specialNumber = 1;
    constructor() {
        owner = msg.sender;
    }
    function restrictedAction(uint256 num)  external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(0x40, 0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}
```
From the example above, we can see that we get a gas saving of over 300 gas when reverting with the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.


```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

107  require(!isApprovedHolder(holder), "holder already approved");

117  require(isApprovedHolder(holder), "holder not approved");

132  require(!LockedForDistribution, "distribution in progress")

134  require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );

152  require(
            !_isPastMinDistributionPeriod(),
            "must distribute before minting or burning"
        );

199  require(numDistributions > 0, "must process at least 1 distribution");

201  require(
                _isPastMinDistributionPeriod(),
                "MinDistributionPeriod not met"
            );

258  require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );

287  require(
            LockedForDistribution,
            "cannot end distribution when not locked"
        );

360  require(!LockedForDistribution, "cannot withdraw during distribution");

397  require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );

431  require(true, "unable to find released NFT in ManagedNFTs");

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L107

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

112  require(
            newErc20s.length == newAmounts.length,
            "threshold values must have the same length"
        );

136  require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );

157   require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );

180   require(result, "unsuccessful withdrawal");

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L112-L115

```solidity
file: main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol

25   require(
            ERC721(this).ownerOf(tokenId) == _msgSender(),
            "OwnableApprovable: caller is not the owner"
        );

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L25-L28

## [G-06] Don’t make variables public unless necessary

Issue Description - Making variables public comes with some overhead costs that can be avoided in many cases. A public variable implicitly creates a public getter function of the same name, increasing the contract size.

Proposed Optimization - Only mark variables as public if their values truly need to be readable by external contracts/users. Otherwise, use private or internal visibility.

Estimated Gas Savings - The savings from avoiding unnecessary public variables are small per transaction, around 5-10 gas. However, this adds up over many transactions targeting a contract with public state variables that don’t need to be public.

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

54  uint256 public constant Version = 1;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L54

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

46  uint256 public constant Version = 1;

53  uint256 public constant AccountId = 1;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L46

## [G-07] Free Up Unused Storage

Deleting your unused variables helps free up space and earns a gas refund. Deleting unused variables has the same effect as reassigning the value type with its default value, such as the integer's default value of 0, or the address zero for addresses.

```solidity
//Using delete keyword
delete myVariable;

//Or assigning the value 0 if integer
myInt = 0;

```
Mappings, however, are unaffected by deletion, as the keys of mappings may be arbitrary and are generally unknown. Therefore, if you delete a struct, all of its members that are not mappings will reset and also recurse into its members. However, individual keys and the values they relate to can be removed.

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
 
266  delete erc20EntitlementPerUnit;

291  delete erc20EntitlementPerUnit;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L266

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

117   delete thresholdErc20s;

118   delete thresholdAmounts;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L117

## [G-08] Shorten arrays with inline assembly

Issue Description - When shortening an array in Solidity, it creates a new shorter array and copies the elements over. This wastes gas by duplicating storage.

Proposed Optimization - Use inline assembly to shorten the array in place by changing its length slot, avoiding the need to copy elements to a new array.

Estimated Gas Savings - Shortening a length-n array avoids ~n SSTORE operations to copy elements. Benchmarking shows savings of 5000-15000 gas depending on original length.

```solidity
function shorten(uint[] storage array, uint newLen) internal {
  assembly {
    sstore(array_slot, newLen)
  }
}
// Rather than:
function shorten(uint[] storage array, uint newLen) internal {
  uint[] memory newArray = new uint[](newLen);
  for(uint i = 0; i < newLen; i++) {
    newArray[i] = array[i];
  }
  delete array;
  array = newArray;
}
```
Using inline assembly allows shortening arrays without copying elements to a new storage slot, providing significant gas savings.

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol

174  uint256[] memory amounts = new uint256[](erc20s.length);

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L174

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
 
217  uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L217-L219

## [G-09] Using bytes32 is cheaper than using string. Storing and manipulating data in bytes32 is more 

gas-efficient than using string, which involves dynamic storage allocation. bytes32 is a fixed-size type, whereas string can have variable length, resulting in higher gas costs.

```solidity
file: blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

457  string memory _name,

458  string memory _symbol,

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L457

## [G-10] Mappings not used externally/internally can be marked private

The below mappings from the LiquidInfrastructureERC20.sol contract can be marked private since they’re not used by any other contracts.

```solidity
file: main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

65    mapping(address => bool) public HolderAllowlist;

```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L65