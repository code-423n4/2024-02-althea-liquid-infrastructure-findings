# Quality Assurance Report

### Low-Severity

## [L-01] Use inbuilt safeTransferFrom instead of transferFrom to transfer ERC721 tokens.

It implements check onERC721Received when transferring tokens to contracts. Which ensure that receiver contracts can handle NFTs.

```solidity

418: nft.transferFrom(address(this), to, nft.AccountId());
```

[L-418](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L418C8-L418C62)

## [L-02] `distribute` function of `LiquidInfrastructureERC20.sol` will revert if array `erc20EntitlementPerUnit` length is less than `distributableERC20s` array length.

Since both uses `distributableERC20s.length` to access their elements 

```solidity
  for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }

```

[Code](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214C7-L228C1)

## [L-03] Every ERC20 not returns true on success so this can break the logic when erc20 don't return bool on success/failure like USDT protocol.

It is stated in ReadMe that protocol will use usdt also. So whenever function like transfer will be called on USDT contract then it will not return true on success resulting the revert of `distribute` function of `LiquidInfrastructureERC20.sol`.

```solidity
File : liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

198:    function distribute(uint256 numDistributions) public nonReentrant {
...
224:            if (toDistribute.transfer(recipient, entitlement)) {
225:                     receipts[j] = entitlement;
226:             }

```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224C1-L226C22

### Recommended mitigation :

Use openzeppelin SafeERC20 library. Use safeTransfer of that library instead of transfer. So whenever any token return bool or revert on failure it can handle both situations. Since it uses low level call to call the ERC20 contract.
