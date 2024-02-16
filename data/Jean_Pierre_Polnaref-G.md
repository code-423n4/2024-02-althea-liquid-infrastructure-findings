# c4udit Report

## Files analyzed
- 2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol
- 2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol
- 2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/OwnableApprovableERC721.sol

## Issues found

### Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g001---dont-initialize-variables-with-default-value)

#### Findings:
```
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::171 => for (uint i = 0; i < holders.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::220 => for (uint j = 0; j < distributableERC20s.length; j++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::271 => for (uint i = 0; i < distributableERC20s.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::421 => for (uint i = 0; i < ManagedNFTs.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::467 => for (uint i = 0; i < _approvedHolders.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::120 => for (uint i = 0; i < newErc20s.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::175 => for (uint i = 0; i < erc20s.length; i++) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Cache Array Length Outside of Loop

#### Impact
Issue Information: [G002](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g002---cache-array-length-outside-of-loop)

#### Findings:
```
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::171 => for (uint i = 0; i < holders.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::174 => holders[i] = holders[holders.length - 1];
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::185 => uint256 num = holders.length;
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::187 => distribute(holders.length);
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::210 => holders.length
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::218 => distributableERC20s.length
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::220 => for (uint j = 0; j < distributableERC20s.length; j++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::234 => if (nextDistributionRecipient == holders.length) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::245 => if (totalSupply() == 0 || holders.length == 0) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::265 => if (erc20EntitlementPerUnit.length > 0) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::271 => for (uint i = 0; i < distributableERC20s.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::352 => withdrawFromManagedNFTs(ManagedNFTs.length);
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::368 => ManagedNFTs.length
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::382 => if (nextWithdrawal == ManagedNFTs.length) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::421 => for (uint i = 0; i < ManagedNFTs.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::425 => ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::467 => for (uint i = 0; i < _approvedHolders.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::113 => newErc20s.length == newAmounts.length,
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::114 => "threshold values must have the same length"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::120 => for (uint i = 0; i < newErc20s.length; i++) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::174 => uint256[] memory amounts = new uint256[](erc20s.length);
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::175 => for (uint i = 0; i < erc20s.length; i++) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G003](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g003---use--0-instead-of--0-for-unsigned-integer-comparison)

#### Findings:
```
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::186 => if (num > 0) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::199 => require(numDistributions > 0, "must process at least 1 distribution");
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::265 => if (erc20EntitlementPerUnit.length > 0) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::178 => if (balance > 0) {
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Long Revert Strings

#### Impact
Issue Information: [G007](https://github.com/byterocket/c4-common-issues/blob/main/0-Gas-Optimizations.md#g007---long-revert-strings)

#### Findings:
```
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::4 => import "@openzeppelin/contracts/utils/math/Math.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::5 => import "@openzeppelin/contracts/access/Ownable.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::6 => import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::7 => import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::8 => import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::9 => import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::10 => import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::136 => "receiver not approved to hold the token"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::154 => "must distribute before minting or burning"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::199 => require(numDistributions > 0, "must process at least 1 distribution");
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::260 => "cannot begin distribution when already locked"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::289 => "cannot end distribution when not locked"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::360 => require(!LockedForDistribution, "cannot withdraw during distribution");
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::399 => "this contract does not own the new ManagedNFT"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::431 => require(true, "unable to find released NFT in ManagedNFTs");
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::4 => import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::5 => import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::56 => * Constructs the underlying ERC721 with a URI like "althea://liquid-infrastructure-account/{accountName}", and
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::67 => "althea://liquid-infrastructure-account/",
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::114 => "threshold values must have the same length"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::138 => "caller is not the owner of the Account token and is not approved either"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::159 => "caller is not the owner of the Account token and is not approved either"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/OwnableApprovableERC721.sol::4 => import "@openzeppelin/contracts/utils/Context.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/OwnableApprovableERC721.sol::5 => import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/OwnableApprovableERC721.sol::27 => "OwnableApprovable: caller is not the owner"
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/OwnableApprovableERC721.sol::40 => revert("OwnableApprovable: caller is not owner nor approved");
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)

### Unsafe ERC20 Operation(s)

#### Impact
Issue Information: [L001](https://github.com/byterocket/c4-common-issues/blob/main/2-Low-Risk.md#l001---unsafe-erc20-operations)

#### Findings:
```
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::224 => if (toDistribute.transfer(recipient, entitlement)) {
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol::418 => nft.transferFrom(address(this), to, nft.AccountId());
2024-02-althea-liquid-infrastructure/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol::179 => bool result = IERC20(erc20).transfer(destination, balance);
```
#### Tools used
[c4udit](https://github.com/byterocket/c4udit)
