# [L-01] Duplicate Entries Allowed in `addManagedNFT` Function

## Summary

The `LiquidInfrastructureERC20` contract contains a function, `addManagedNFT`, which is responsible for adding new NFT contracts to the `ManagedNFTs` array. A critical issue has been identified where this function does not check for duplicates before adding a new entry. As a result, the same NFT contract can be added multiple times to the `ManagedNFTs` array, leading to potential inefficiencies and logical errors in the contract's operation.

## Impacts

1. **Inefficient Storage**: Duplicate entries in the `ManagedNFTs` array consume unnecessary storage space on the blockchain, leading to increased costs and inefficiency.
2. **Increased Gas Costs**: Contract functions that iterate over the `ManagedNFTs` array will incur higher gas costs due to processing duplicate NFT contracts.
3. **Incorrect Logic Execution**: Logic that depends on the uniqueness of NFT contracts within the `ManagedNFTs` array may operate incorrectly if duplicates are present, potentially affecting the contract's intended behavior.
4. **Potential for Exploitation**: The contract includes mechanisms to deposit all token balances into the custody of `LiquidInfrastructureERC20` contract which is tied to the `ManagedNFTs` array, the presence of duplicates could be cause unexpected reverts.

## Code Snippet

The issue arises from the `addManagedNFT` function, which currently lacks duplicate checks:

```solidity
    function addManagedNFT(address nftContract) public onlyOwner {
        // There should be a check here to prevent duplicates
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394

## Recommendation

**Implement Duplicate Checks**: Update the `addManagedNFT` function to include a check for the existence of the NFT contract address in the `ManagedNFTs` array before adding it.

# [L-02] `addManagedNFT`, `releaseManagedNFT`, and `setDistributableERC20s` functions in `LiquidInfrastructureERC20` contract don't check if a withdrawal or distribution is in progress before updating

## Summary
The `addManagedNFT`, `releaseManagedNFT`, and `setDistributableERC20s` functions in the LiquidInfrastructureERC20 contract do not check whether withdrawals from the ManagedNFTs collection or distributions are in progress before allowing modifications. This could lead to inconsistent state if these functions are called while withdrawals or distributions are ongoing.

## Impact
If these functions are called while withdrawals or distributions are in progress, it could lead to unexpected behavior and potential loss of funds for token holders. For example, adding or releasing a ManagedNFT or changing the distributable ERC20s list during an ongoing withdrawal or distribution could result in incorrect token balances or distributions.

## Proof of Concept
Link to affected code

1. https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394
2. https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413
3. https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441

**Steps to Reproduce:**
1. Initiate a withdrawal from the `ManagedNFTs` collection using `withdrawFromManagedNFTs` function or do a distribution using `distribute()`.
2. While the withdrawal/distribution is in progress, call the `addManagedNFT`, `releaseManagedNFT`, or `setDistributableERC20s` function to modify the contract state.

**Expected Result:** The contract should prevent modifications to the `ManagedNFTs` collection or distributable ERC20s list while withdrawals or distributions are in progress.

**Actual Result:** The contract allows modifications to the `ManagedNFTs` collection or distributable ERC20s list while withdrawals or distributions are in progress, potentially leading to inconsistent state.

## Tools Used
Manual Review

## Recommended Mitigation Steps
Add modifiers to the `addManagedNFT`, `releaseManagedNFT`, and `setDistributableERC20s` functions to check whether withdrawals from the `ManagedNFTs` collection or distributions are in progress before allowing modifications.


# [NC-01] Redundant Code in `LiquidInfrastructureERC20` Contract

## Summary
The code snippet `if (erc20EntitlementPerUnit.length > 0) { delete erc20EntitlementPerUnit; }` in the `LiquidInfrastructureERC20` contract is redundant. This code checks if `erc20EntitlementPerUnit` has a length greater than zero before deleting it, but the deletion is already performed in the `_endDistribution()` function. Since `erc20EntitlementPerUnit` can only be modified in the `_beginDistribution()` function and is always deleted in `_endDistribution()`, this check is unnecessary.

## Impact
The redundant check does not impact the functionality of the contract but adds unnecessary complexity to the code. Removing this check will simplify the codebase and make it easier to maintain.

## Proof of Concept
[Link to affected code](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L265C8-L267C10)

```solidity
    if (erc20EntitlementPerUnit.length > 0) { // @audit redundant
        delete erc20EntitlementPerUnit;
    }
```

## Tools Used
Manual Review

## Recommended Mitigation Steps
Remove the redundant check and deletion of `erc20EntitlementPerUnit` in the `LiquidInfrastructureERC20` contract, as it is already handled in the `_endDistribution()` function. This will simplify the code and improve readability.

# [NC-02] Redundant Ownership Check in `LiquidInfrastructureNFT`

## Description
The `withdrawBalances` and `withdrawBalancesTo` functions in the `LiquidInfrastructureNFT` contract contains a redundant ownership check using a `require` statement. This check is already encapsulated by the `onlyOwnerOrApproved` modifier provided by the `OwnableApprovableERC721` contract. The current implementation is as follows:

```solidity
    function withdrawBalances(address[] calldata erc20s) public virtual {
@-->    require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }

    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) public virtual {
@-->    require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        _withdrawBalancesTo(erc20s, destination);
    }
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L135C1-L162C6

## Recommendation
To improve code readability and adhere to the DRY principle, it is recommended to use the `onlyOwnerOrApproved` modifier directly, which simplifies the function and removes the explicit require statement. 

```diff
-   function withdrawBalances(address[] calldata erc20s) public virtual {
+   function withdrawBalances(address[] calldata erc20s) public virtual onlyOwnerOrApproved(AccountId) {
-       require(
-           _isApprovedOrOwner(_msgSender(), AccountId),
-           "caller is not the owner of the Account token and is not approved either"
-       );
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }

    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
-   ) public virtual {
+   ) public virtual onlyOwnerOrApproved(AccountId) {
-       require(
-           _isApprovedOrOwner(_msgSender(), AccountId),
-           "caller is not the owner of the Account token and is not approved either"
-       );
        _withdrawBalancesTo(erc20s, destination);
    }
```