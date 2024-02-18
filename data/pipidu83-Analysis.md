# Table of Contents

- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
- [Findings](#findings)
  - [High](#high)
    - [\[H-1\] USDTs will be locked into the LiquidInfrastructureNFT contract forever](#h-1-usdts-will-be-locked-into-the-liquidinfrastructurenft-contract-forever)
    - [\[H-2\] Looping through the unbounded holders array means LiquidInfrastructureERC20::\_afterTokenTransfer will run out of gas for a large number of holders](#h-2-looping-through-the-unbounded-holders-array-means-liquidinfrastructureerc20_aftertokentransfer-will-run-out-of-gas-for-a-large-number-of-holders)
    - [\[H-3\] Looping through the unbounded ManagedNFTs array means LiquidInfrastructureERC20::releaseManagedNFT will run out of gas for a large number of managed NFTs](#h-3-looping-through-the-unbounded-managednfts-array-means-liquidinfrastructureerc20releasemanagednft-will-run-out-of-gas-for-a-large-number-of-managed-nfts)
    - [\[H-4\] A Managed NFT can be added more than once, potentially breaking protocol economics](#h-4-a-managed-nft-can-be-added-more-than-once-potentially-breaking-protocol-economics)
  - [Non-Critical](#non-critical)
    - [\[NC-1\] LiquidInfrastructureERC20::MinDistributionPeriod (contracts/LiquidInfrastructureERC20.sol#76) should be immutable](#nc-1-liquidinfrastructureerc20mindistributionperiod-contractsliquidinfrastructureerc20sol76-should-be-immutable)
    - [\[NC-2\] Use named imports to add readability and safety](#nc-2-use-named-imports-to-add-readability-and-safety)
    - [\[NC-3\] Use a more coherent design of modifiers in the OwnableApprovableERC721 contract](#nc-3-use-a-more-coherent-design-of-modifiers-in-the-ownableapprovableerc721-contract)
    - [\[NC-4\] Place the constructor at the top of the contract to improve readability](#nc-4-place-the-constructor-at-the-top-of-the-contract-to-improve-readability)
    - [\[NC-5\] The `nonReentrant` `modifier` should occur before all other modifiers](#nc-5-the-nonreentrant-modifier-should-occur-before-all-other-modifiers)
    - [\[NC-6\] Functions not used internally could be marked external](#nc-6-functions-not-used-internally-could-be-marked-external)
    - [\[NC-7\] Missing checks for `address(0)` when assigning values to address state variables](#nc-7-missing-checks-for-address0-when-assigning-values-to-address-state-variables)
    - [\[NC-8\] Non conformance to solidity naming conventions](#nc-8-non-conformance-to-solidity-naming-conventions)
  - [Gas](#gas)
    - [\[G-1\] Loops should use cached array length instead of referencing the 'length' member of a storage array](#g-1-loops-should-use-cached-array-length-instead-of-referencing-the-length-member-of-a-storage-array)
    - [\[G-2\] holders.length is cached but unnecessarily recomputed in the distributeToAllHolders function](#g-2-holderslength-is-cached-but-unnecessarily-recomputed-in-the-distributetoallholders-function)
    - [\[G-3\] Use ++increment instead of increment++ in loops to save gas](#g-3-use-increment-instead-of-increment-in-loops-to-save-gas)
    - [\[G-4\] Use if-else statements and custom errors instead of require() to save gas](#g-4-use-if-else-statements-and-custom-errors-instead-of-require-to-save-gas)


# Protocol Summary

The Liquid Infrastructure protocol allows for the tokenization of Real World Assets (RWAs) and enables the distribution of revenue to holders of these assets in proportion of their holdings.

The `LiquidInfrastructureERC20` contract is responsible for the distribution of the above mentioned revenue and the `LiquidInfrastructureNFT` represents the tokenized assets.

# Risk Classification

We use the below matrix based on Impact and Likelihood to determine the severity of each issue detailed in this analysis report.

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H      | M   |
| Likelihood | Medium | H      | M      | M   |
|            | Low    | M      | M      | L   |

# Audit Details 


## Scope 

Their is 377 SLOC in scope, split in the 3 files below.

```
./liquid-infrastructure/
-- LiquidInfrastructureERC20.sol
-- LiquidInfrastructureNFT.sol
-- OwnableApprovableERC721.sol
```

## Roles

Owner - Owner of the NFT. His rights cover the setting of thresholds to determine how much of each token should be left in the Liquid Account, the withdrawal of balances, initiating the account recovery, approving or disapproving ERC20 holders, minting and distribution tokens, as well as the addition and release of managed NFTs.

# Executive Summary

Overall we find that the structure of the contracts make sense for the protocol. 
We found a few high severity vulnerabilies that could endanger the protocol's viability. Additionally, more attention could be paid to the finish to produce high quality production-ready code. This includes the range of comments that we listed in the "Non Critical" section, including but not limited to using named imports, and making sure the design and ordering of functions is coherent and easy to read for external users and auditors. 

Although not critical and with no severe consequence on the overall good-functioning of the protocol, we advise against the definition of functions that "may exceed the block gas limit" or "may trigger out of gas errors" as per the natspecs of the `LiquidInfrastructureERC20::burnFromAndDistribute`, `LiquidInfrastructureERC20::distributeToAllHolders`, `LiquidInfrastructureERC20::mintAndDistribute` and `LiquidInfrastructureERC20::burnAndDistribute` functions. We believe the protocol should instead opt for "cleaner" versions of these functions that cap the number of transfers/transactions to a maximum value to perform these actions in batches that will not run out of gas. That will improve overall user experience of the protocol and prevent the use of functions that may revert depending on the conditions.

Throughout the protocol, it would be wise to stick to the Checks-Effects-Interactions pattern as a safer way to execute logic inside smart contracts. In particular, this will mitigate any risk of reeantrancy in functions transferring tokens. We notice occurences of this in `LiquidInfrastructureERC20::burnAndDistribute`, `LiquidInfrastructureERC20::burnFromAndDistribute`, `LiquidInfrastructureERC20::distribute`, `LiquidInfrastructureERC20::mintAndDistribute` and `LiquidInfrastructureERC20::withdrawFromManagedNFTs`.

We notice a high centralization of the protocol through the wide range of rights granted to the owner role. This could be mitigated by implementing a voting mechanism with some decisions being taken by users of the protocol (e.g. the addition of Managed NFTs).

## Issues found

| Severity | Number of issues found |
| -------- | ---------------------- |
| High     | 4                      |
| Medium   | 0                      |
| Low      | 0                      |
| NC       | 8                      |
| Gas      | 4                      |
| Total    | 15                     |

# Findings

## High

### [H-1] USDTs will be locked into the LiquidInfrastructureNFT contract forever

**Impact**

As mentioned in Liquid Infrastructure documentation:

"Liquid Infrastructure is expected to use ERC20 stablecoins like USDC, Dai, or USDT as revenue tokens. The protocol is expected to vet ERC20s and opt-in to their use to avoid issues with non-standard tokens (e.g. rebasing tokens)"

However, USDT is not compliant with the ERC20 token standard and does not return a boolean on `transfer`. This is an issue as the `LiquidInfrastructureNFT` contract checks for this return value when calling `LiquidInfrastructureNFT::_withdrawBalancesTo`, which is a necessary function to withdraw funds from the contract.

```javascript
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```

The `require` statement above will always revert when the function is called with USDT because `result` can never be `true`. This means that this function will be unusable with USDT tokens.

As this vulnerability could result in the loss of funds that are being locked forever, we mark it as HIGH.

**Proof of Concept**

Let's assume that the `LiquidInfrastructureNFT` contract holds 1000 USDT for the sake of this proof of concept.

1. Owner calls the `LiquidInfrastructureNFT::withdrawBalances` function with the `erc20s` parameter set to `[address(USDT)]`

```javascript
    function withdrawBalances(address[] calldata erc20s) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }
```

2. `LiquidInfrastructureNFT::_withdrawBalancesTo` is entered with parmeters `[address(USDT)]` and `destination == 1`

```javascript
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```

3. `IERC20(address(USDT)).transfer(destination, balance)` does not return any value as USDT does not comply with the ERC20 standard. This means `result` will be void and the below line will fail

```javascript
    require(result, "unsuccessful withdrawal");
```

This means this function will always revert when used with USDT and the owner will never be able to transfer USDT out of the contract, leading to the loss of these funds.

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

We recommend using OpenZeppelin's [`SafeERC20`](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/utils/SafeERC20.sol) library and its `safeTransfer` function that will handle the return value check properly as well as non compliant ERC20 tokens like USDT.

```diff
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
-               bool result = IERC20(erc20).transfer(destination, balance);
-               require(result, "unsuccessful withdrawal");
+               IERC20(erc20).safeTransfer(destination, balance);
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```

### [H-2] Looping through the unbounded holders array means LiquidInfrastructureERC20::_afterTokenTransfer will run out of gas for a large number of holders

**Impact**

Let us take a look at the `LiquidInfrastructureERC20::_afterTokenTransfer` function below

```javascript
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {

            // @ audit can this be DoS-ed? i.e. can we add a large range of holders??? probably
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

Whenever the `if (!stillHolding)` is met, we will loop through the `holders` array wich is defined as an unbounded array of addresses

```javascript
    address[] private holders;
```

This means that the gas cost required to run this function is an increasing function of the `holders` array's length. This in turn means that for a large enough number of addresses inside the `holders` array, the `LiquidInfrastructureERC20::_afterTokenTransfer` will run out of gas and revert.

As the `LiquidInfrastructureERC20::_afterTokenTransfer` function will be called after any transfer of tokens (including minting or burning), this DoS will severely affect the protocol key functionalities. We then mark this vulnerability as HIGH.

Moreover, as stated in the audit documentation:

"DoS attacks against the LiquidInfrastructureERC20 are of significant concern, particularly if there is a way to permanently trigger out of gas errors"

**Proof of Concept**

Let's assume a very large number N of `holders`.

1. `transfer` is called from `Bob` to `Alice`, with `amount==this.balanceOf(Bob)`
2. `LiquidInfrastructureERC20::_afterTokenTransfer` is called at the end of the `transfer`, as required by the ERC20 standard.
3. `stillHolding` will be `false` as `this.balanceOf(from)` will then be `0` as `transfer` was called with his whole balance. This means we enter the `if (!stillHolding)` condition.
4. For a very large number N of `holders`, the below loop will run out of gas and the whole transaction will revert.

```javascript
    for (uint i = 0; i < holders.length; i++) {
        if (holders[i] == from) {
            // Remove the element at i by copying the last one into its place and removing the last element
            holders[i] = holders[holders.length - 1];
            holders.pop();
        }
    }
```

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

To mitigate this issue and avoid Denial-of-Service, we could for example use a mapping of `holders` to their balance and set this value to `0` for the concerned `holder` when they no longer hold any balance. 
Alternatively we could set a `MAX_HOLDERS` constant variable to cap the number of holders for a given asset but this might restrict the capabilities of the protocol.

### [H-3] Looping through the unbounded ManagedNFTs array means LiquidInfrastructureERC20::releaseManagedNFT will run out of gas for a large number of managed NFTs

**Impact**

Let us take a look at the `LiquidInfrastructureERC20::releaseManagedNFT` function below

```javascript
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```

In this function, we loop through the `ManagedNFTs` array wich is defined as an unbounded array of addresses

```javascript
    address[] public ManagedNFTs;
```

This means that the gas cost required to run this function is an increasing function of the `ManagedNFTs` array's length. This in turn means that for a large enough number of addresses inside the `ManagedNFTs` array, the `LiquidInfrastructureERC20::releaseManagedNFT` will run out of gas and revert.

As the `LiquidInfrastructureERC20::releaseManagedNFT` failure means it will be impossible for the owner to transfer NFTs out of the contract as the function will revert preventing the 

```javascript
    nft.transferFrom(address(this), to, nft.AccountId());
```

line from being executed, this leads to a potential loss of tokens as these NFTs will be locked in the contract forever. Moreover, the owner cannot mitigate it by removing elements from the `ManagedNFTs` array as the very function used to release NFTs and decrease the size of this array is the one being DoS-ed (`LiquidInfrastructureERC20::releaseManagedNFT`).

Moreover, as stated in the audit documentation:

"DoS attacks against the LiquidInfrastructureERC20 are of significant concern, particularly if there is a way to permanently trigger out of gas errors"

**Proof of Concept**

Let's assume a very large number N of `ManagedNFTs`.

1. `releaseManagedNFT` is called from the owner with parameters `NFTcontract` and `Bob`
2. For a very large number N of `ManagedNFTs`, and if `NFTContract` is situated towards the end of the `ManagedNFTs` array (otherwise the `break` statement would save us) the below loop will run out of gas and the whole transaction will revert.

```javascript
    for (uint i = 0; i < ManagedNFTs.length; i++) {
        address managed = ManagedNFTs[i];
        if (managed == nftContract) {
            // Delete by copying in the last element and then pop the end
            ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
            ManagedNFTs.pop();
            break;
        }
    }
```

This means the previous line below will also be reverted and the NFT will not be released and sent back to `Bob`.

```javascript
    nft.transferFrom(address(this), to, nft.AccountId());
```

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

We could set a `MAX_MANAGED_NFTS` constant variable to cap the number of holders for a given asset to ensure the `LiquidInfrastructureERC20::releaseManagedNFT` never runs out of gas. 
We note that this mitigation might not be ideal as compromising the scalability of the protocol to a large number of `ManagedNFTs`.


### [H-4] A Managed NFT can be added more than once, potentially breaking protocol economics

**Impact**

Let's have a look at the `LiquidInfrastructureERC20::addManagedNFT` function below

```javascript
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```

No check is being performed as to wether the NFT we are adding the the `ManagedNFTs` array is already present in the array, meaning the owner could add twice the same NFT to this array.

Upon release of this NFT in the `LiquidInfrastructureERC20::releaseManagedNFT` function, the NFT will be transferred back to is recipient but one occurrence of its address will stay in the `ManagedNFTs` array.

This means the `LiquidInfrastructureERC20::withdrawFromManagedNFTs` could still be called and benefits could be wrongly earned by users from a NFT that is no longer owned by the contract, effectively breaking the economics of the contract.

One could argue that this vulnearability is of MEDIUM severity as the `LiquidInfrastructureERC20::addManagedNFT` function is protected by the `onlyOwner` modifier. However, we believe it has a high likelihood of happening by a mistake or a social engineering attack and it would severely break the economics of the protocol. We then mark this vulnearability as HIGH.

**Proof of Concept**

1. Owner calls `LiquidInfrastructureERC20::addManagedNFT` with `nftContract==NFTa`, with `NFTa` the address of an NFT that they own.
2. `NFTa` gets added to the `ManagedNFTs` array via the below line

```javascript
    ManagedNFTs.push(nftContract);
```

3. Owner calls `LiquidInfrastructureERC20::addManagedNFT` with `nftContract==NFTa` a second time
4. `NFTa` gets added to the `ManagedNFTs` array a second time.
5. Owner calls `LiquidInfrastructureERC20::releaseManagedNFT` for `NFTa` and a given recipient `to`
6. `NFTa` is transferred to the recipient and not owned by the contract anymore after execting the line

```javascript
    nft.transferFrom(address(this), to, nft.AccountId());
```

However, in the snippet below, only one occurrence of `NFTa` will be removed from the `ManagedNFTs` array because of the `break` statement.

```javascript
    for (uint i = 0; i < ManagedNFTs.length; i++) {
        address managed = ManagedNFTs[i];
        if (managed == nftContract) {
            // Delete by copying in the last element and then pop the end
            ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
            ManagedNFTs.pop();
            break;
        }
    }
```

7. owner cannot call `LiquidInfrastructureERC20::releaseManagedNFT` a second time as the line 

```javascript
    nft.transferFrom(address(this), to, nft.AccountId());
```

will revert (as `NFTa` was already transferred back to the recipient). This means `NFTa` will still be in the `ManagedNFTs` contract, which is a severe issue as it means we will still be able to call the `LiquidInfrastructureERC20::withdrawFromManagedNFTs` function of the same contract, withdrawing ERC20 balances linked to a NFT which is not owned by this contract anymore.

**Tools Used**

Manual Review / Visual Studio

**Recommended Mitigation Steps**

The mitigation for this vulnerability can be to define a `mapping(address => bool) public isManaged` mapping to check if a NFT is managed. We can then add a check inside the `LiquidInfrastructureERC20::addManagedNFT` to avoid adding the same NFT more than once.

```diff
    function addManagedNFT(address nftContract) public onlyOwner {
+       require(!(isManaged(nftContract)), "NFT already managed");        
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```


## Non-Critical

### [NC-1] LiquidInfrastructureERC20::MinDistributionPeriod (contracts/LiquidInfrastructureERC20.sol#76) should be immutable

More information can be found in https://github.com/crytic/slither/wiki/Detector-Documentation#state-variables-that-could-be-declared-immutable

### [NC-2] Use named imports to add readability and safety

- Found in contracts/LiquidInfrastructureERC20.sol Line: 4

```solidity
import "@openzeppelin/contracts/utils/math/Math.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "./LiquidInfrastructureNFT.sol";
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 4

```solidity
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "./OwnableApprovableERC721.sol";
```

- Found in contracts/OwnableApprovableERC721.sol Line: 4

```solidity
import "@openzeppelin/contracts/utils/Context.sol";
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```

### [NC-3] Use a more coherent design of modifiers in the OwnableApprovableERC721 contract

We chose the design of the `OwnableApprovableERC721::onlyOwnerOrApproved` which has the added advantage of being more gas efficient.

```diff
    /**
     * @dev Throws if called by any account other than the owner.
     */
    modifier onlyOwner(uint256 tokenId) {
-        require(
-            ERC721(this).ownerOf(tokenId) == _msgSender(),
-            "OwnableApprovable: caller is not the owner"
-        );
+        if (ERC721(this).ownerOf(tokenId) == _msgSender()) {
        _;
+        } else {
+            revert("OwnableApprovable: caller is not the owner");
+        }
    }

    /**
     * @dev Throws if called by any account other than the owner or someone approved by the owner
     */
     // @ audit why dont use same structure as above i.e. require??
    modifier onlyOwnerOrApproved(uint256 tokenId) {
        // Get approval directly from ERC721's internal method
        if (_isApprovedOrOwner(_msgSender(), tokenId)) {
            _;
        } else {
            revert("OwnableApprovable: caller is not owner nor approved");
        }
    }
```

### [NC-4] Place the constructor at the top of the contract to improve readability

In the `LiquidInfrastructureERC20` contract, the `constructor` is placed at the end of the contract, which is harmful to overall readability of the contract and lacks coherence with the `LiquidInfrastructureNFT` contract, whose `constructor` is defined at the top of the contract.

### [NC-5] The `nonReentrant` `modifier` should occur before all other modifiers

This is a best-practice to protect against reentrancy in other modifiers

- Found in contracts/LiquidInfrastructureERC20.sol Line: 326

	```solidity
	    ) public onlyOwner nonReentrant {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 422

	```solidity
	    ) public onlyOwner nonReentrant {
	```

### [NC-6] Functions not used internally could be marked external


- Found in contracts/LiquidInfrastructureERC20.sol Line: 107

	```solidity
	    function approveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 117

	```solidity
	    function disapproveHolder(address holder) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 308

	```solidity
	    function mintAndDistribute(
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 336

	```solidity
	    function burnAndDistribute(uint256 amount) public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 349

	```solidity
	    function burnFromAndDistribute(address account, uint256 amount) public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 356

	```solidity
	    function withdrawFromAllManagedNFTs() public {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 400

	```solidity
	    function addManagedNFT(address nftContract) public onlyOwner {
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 419

	```solidity
	    function releaseManagedNFT(
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 447

	```solidity
	    function setDistributableERC20s(
	```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 85

	```solidity
	    function getThresholds()
	```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 109

	```solidity
	    function setThresholds(
	```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 138

	```solidity
	    function withdrawBalances(address[] calldata erc20s) public virtual {
	```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 156

	```solidity
	    function withdrawBalancesTo(
	```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 207

	```solidity
	    function recoverAccount() public virtual onlyOwner(AccountId) {
	```


### [NC-7] Missing checks for `address(0)` when assigning values to address state variables

Assigning values to address state variables without checking for `address(0)`.

- Found in contracts/LiquidInfrastructureERC20.sol Line: 450

	```solidity
	        distributableERC20s = _distributableERC20s;
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 471

	```solidity
	        ManagedNFTs = _managedNFTs;
	```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 480

	```solidity
	        distributableERC20s = _distributableErc20s;
	```


### [NC-8] Non conformance to solidity naming conventions

Consider using mixedCase for the naming of variables for better readability. More information can about this non-critical issue can be found in https://github.com/crytic/slither/wiki/Detector-Documentation#conformance-to-solidity-naming-conventions


- Found in contracts/LiquidInfrastructureERC20.sol Line: 61

```solidity
      address[] public ManagedNFTs;
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 66

```solidity
    mapping(address => bool) public HolderAllowlist;
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 71

```solidity
    uint256 public LastDistribution;
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 76

```solidity
    uint256 public MinDistributionPeriod;
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 81

```solidity
    bool public LockedForDistribution;
```

## Gas

### [G-1] Loops should use cached array length instead of referencing the 'length' member of a storage array

More information in https://github.com/crytic/slither/wiki/Detector-Documentation#cache-array-length

- Found in contracts/LiquidInfrastructureERC20.sol Line: 174

```solidity
    for (uint i = 0; i < holders.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 224

```solidity
    for (uint j = 0; j < distributableERC20s.length; j++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 275

```solidity
    for (uint i = 0; i < distributableERC20s.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 427

```solidity
    for (uint i = 0; i < ManagedNFTs.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 474

```solidity
    for (uint i = 0; i < _approvedHolders.length; i++) {
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 121

```solidity
    for (uint i = 0; i < newErc20s.length; i++) {
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 178

```solidity
    for (uint i = 0; i < erc20s.length; i++) {
```

### [G-2] holders.length is cached but unnecessarily recomputed in the distributeToAllHolders function

```diff

    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
-            distribute(holders.length);
+            distribute(num);
        }
    }
```

### [G-3] Use ++increment instead of increment++ in loops to save gas

- Found in contracts/LiquidInfrastructureERC20.sol Line: 174

```solidity
    for (uint i = 0; i < holders.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 218

```solidity
    for (i = nextDistributionRecipient; i < limit; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 224

```solidity
    for (uint j = 0; j < distributableERC20s.length; j++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 275

```solidity
    for (uint i = 0; i < distributableERC20s.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 275

```solidity
    for (i = nextWithdrawal; i < limit; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 427

```solidity
    for (uint i = 0; i < ManagedNFTs.length; i++) {
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 427

```solidity
    for (uint i = 0; i < _approvedHolders.length; i++) {
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 121

```solidity
    for (uint i = 0; i < newErc20s.length; i++) {
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 178

```solidity
    for (uint i = 0; i < erc20s.length; i++) {
```

### [G-4] Use if-else statements and custom errors instead of require() to save gas 

- Found in contracts/LiquidInfrastructureNFT.sol Line: 112

```diff
-        require(
-            newErc20s.length == newAmounts.length,
-            "threshold values must have the same length"
-        );
+        if (!(newErc20s.length == newAmounts.length)) {
+            LiquidInfrastructureNFT__inputLengthError();
+        }
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 138

```solidity
       require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
```

- Found in contracts/LiquidInfrastructureNFT.sol Line: 159

```solidity
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
```


- Found in contracts/LiquidInfrastructureNFT.sol Line: 183

```solidity
    require(result, "unsuccessful withdrawal");
```

- Found in contracts/OwnableApprovableERC721.sol Line: 24

```solidity
        require(
            ERC721(this).ownerOf(tokenId) == _msgSender(),
            "OwnableApprovable: caller is not the owner"
        );
```


- Found in contracts/LiquidInfrastructureERC20.sol Line: 108

```solidity
    require(!isApprovedHolder(holder), "holder already approved");
```


- Found in contracts/LiquidInfrastructureERC20.sol Line: 118

```solidity
    require(isApprovedHolder(holder), "holder not approved");
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 133

```solidity
    require(!LockedForDistribution, "distribution in progress");
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 135

```solidity
    require(
        isApprovedHolder(to),
        "receiver not approved to hold the token"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 153

```solidity
    require(
        !_isPastMinDistributionPeriod(),
        "must distribute before minting or burning"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 205

```solidity
    require(
        _isPastMinDistributionPeriod(),
        "MinDistributionPeriod not met"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 262

```solidity
    require(
        !LockedForDistribution,
        "cannot begin distribution when already locked"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 291

```solidity
    require(
        LockedForDistribution,
        "cannot end distribution when not locked"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 365

```solidity
    require(!LockedForDistribution, "cannot withdraw during distribution");
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 403

```solidity
    require(
        nftOwner == address(this),
        "this contract does not own the new ManagedNFT"
    );
```

- Found in contracts/LiquidInfrastructureERC20.sol Line: 437

```solidity
    require(true, "unable to find released NFT in ManagedNFTs");
```


### Time spent:
12 hours