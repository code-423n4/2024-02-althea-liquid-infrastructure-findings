## [L-1] Constant value `true` passed to `require` function as argument

The `releaseManagedNFT` function of LiquidInfrastructureERC20.sol uses a require statement that with a constant boolean literal. This does not validate anything since the argument is constant `true`

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431
```
File: LiquidInfrastructureERC20.sol

// By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");
```

Recommendation:
Consider implementing the necessary validation with the require statement instead of passing true.


## [L-2] ERC721 inherited twice on `LiquidInfrastructureNFT` contract.

The `LiquidInfrastructureNFT` inherits from ERC721 while its parent contract `OwnableApprovableERC721` also inherit from ERC721.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L33
```
contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
```

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L20
```
abstract contract OwnableApprovableERC721 is Context, ERC721 {
```

Recommendation:
Consider removing the ERC721 on the `LiquidInfrastructureNFT` contract since its parent `OwnableApprovableERC721` already inherited it.

```diff
--   contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
++   contract LiquidInfrastructureNFT is OwnableApprovableERC721 {
```