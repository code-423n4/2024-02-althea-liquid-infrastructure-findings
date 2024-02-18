## MODIFIERS CREATED BUT RARELY USED

**Description:** `OwnableApprovableERC721.sol` is a contract that provides modifiers for access controls. However, these functionalities are not being utilized and new checks are made for functions that have "This function is access controlled" in their natspec. As a result, gas is being used more than needed.

**Impact:** Unnecessary gas expenditure.

**Proof of Concept:**

[Code excerpt](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L133-L142):
```solidity
* @notice This function is access controlled, only the owner or an approved msg.sender may call this function
     */
    function withdrawBalances(address[] calldata erc20s) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        address destination = ownerOf(AccountId);
        _withdrawBalancesTo(erc20s, destination);
    }
```

[Code excerpt](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L151-L162):
```solidity
* @notice This function is access controlled, only the owner or an approved msg.sender may call this function
     */
    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        _withdrawBalancesTo(erc20s, destination);
    }
```

**Recommended Mitigation:** Utilize the modifiers with the indicated functions. All of the functions above have a require check that can be easily swapped with the necessary modifier.