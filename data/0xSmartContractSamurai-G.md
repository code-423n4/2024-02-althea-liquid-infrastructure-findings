0. GAS: `LiquidInfrastructureERC20::distributeToAllHolders()` - L167: Could contribute to out of gas DoS due to use of state variable `holders.length` as parameter for calling method `distribute()`. Should use cached local variable `num` instead.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L167

The code change from `distribute(holders.length`) to `distribute(num)` in the `distributeToAllHolders()` function improves gas efficiency by using a locally cached variable `num` to store the value of `holders.length` and then passing `num` as a parameter to the `distribute()` function. This optimization reduces gas costs by avoiding redundant computation and storage access:
```diff
    /**
     * Performs a distribution to all of the current holders, which may trigger out of gas errors if there are too many holders
     */
    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
-           distribute(holders.length);
+           distribute(num);
        }
    }
```

=======
=======

1. GAS: `LiquidInfrastructureERC20::` - Use `!= 0` instead of `> 0`, to save some gas.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L166

```diff
- if (num > 0) {
+ if (num != 0) {
```
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L179
```diff
- require(numDistributions > 0, "must process at least 1 distribution");
+ require(numDistributions != 0, "must process at least 1 distribution");
```

=======
=======

2. GAS: `LiquidInfrastructureERC20::distribute()` - some much needed gas optimizations for this function.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L171-L208

Recommended gas optimizations:
```diff
    function distribute(uint256 numDistributions) public nonReentrant {
-       require(numDistributions > 0, "must process at least 1 distribution"); 
+       require(numDistributions != 0, "must process at least 1 distribution");
        if (!LockedForDistribution) { 
            require(_isPastMinDistributionPeriod(), "MinDistributionPeriod not met");
            _beginDistribution(); 
        }

+		/// cache some state variables
+		uint256 _nextDistributionRecipient = nextDistributionRecipient;
+		uint256 _holdersLength = holders.length;
-       uint256 limit = Math.min(nextDistributionRecipient + numDistributions, holders.length);
+       uint256 limit = Math.min(_nextDistributionRecipient + numDistributions, _holdersLength);

+		uint256 distributableERC20sLength = distributableERC20s.length;
        uint256 i;
-       for (i = nextDistributionRecipient; i < limit; i++) {
+       for (i = _nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
-               uint256[] memory receipts = new uint256[](distributableERC20s.length);
+               uint256[] memory receipts = new uint256[](distributableERC20sLength);
+				uint256 thisBalanceOfRecipient = this.balanceOf(recipient);
-               for (uint256 j = 0; j < distributableERC20s.length; j++) {
+               for (uint256 j; j < distributableERC20sLength; j++) { 
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
-                   uint256 entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient);
+                   uint256 entitlement = erc20EntitlementPerUnit[j] * thisBalanceOfRecipient;
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }

                emit Distribution(recipient, distributableERC20s, receipts);
            }
        }
        nextDistributionRecipient = i;
+       _nextDistributionRecipient = nextDistributionRecipient;

-       if (nextDistributionRecipient == holders.length) {
+		if (_nextDistributionRecipient == _holdersLength) {
            _endDistribution();
        }
    }
```

=======
=======

3. GAS: Cache the state variable's length value.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L240

Cache the state variable's length value:
```diff
+ uint256 distributableERC20sLength = distributableERC20s.length;
- for (uint256 i = 0; i < distributableERC20s.length; i++) { 
+ for (uint256 i; i < distributableERC20sLength; i++) { 
```

=======
=======

4. GAS: `withdrawFromAllManagedNFTs()` - Cache the state variable and use the cached local variable as the method's parameter.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L309-L311

```diff
    function withdrawFromAllManagedNFTs() public { 
+		uint256 ManagedNFTsLength = ManagedNFTs.length;    
-   	withdrawFromManagedNFTs(ManagedNFTs.length);
+       withdrawFromManagedNFTs(ManagedNFTsLength); 
    }
```
=======
=======

5. GAS: cache the state variables in `withdrawFromManagedNFTs()`.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/3adc34600561077ad4834ee9621060afd9026f06/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L317-L339

```diff
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

+		uint256 _nextWithdrawal = nextWithdrawal;
-       if (nextWithdrawal == 0) {
+       if (_nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

+		uint256 ManagedNFTsLength = ManagedNFTs.length;
-       uint256 limit = Math.min(numWithdrawals + nextWithdrawal, ManagedNFTs.length);
+       uint256 limit = Math.min(numWithdrawals + _nextWithdrawal, ManagedNFTsLength);
        uint256 i;
-       for (i = nextWithdrawal; i < limit; i++) {
+       for (i = _nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(ManagedNFTs[i]);

            (address[] memory withdrawERC20s,) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;
+       _nextWithdrawal = nextWithdrawal;

-       if (nextWithdrawal == ManagedNFTs.length) {
+       if (_nextWithdrawal == ManagedNFTsLength) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```
=======
=======

6. GAS: cache `ManagedNFTs.length`.

```diff
    function releaseManagedNFT(address nftContract, address to) public onlyOwner nonReentrant { 
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId()); 

+		uint256 ManagedNFTsLength = ManagedNFTs.length;
        // Remove the released NFT from the collection 
-       for (uint256 i = 0; i < ManagedNFTs.length; i++) {
+       for (uint256 i; i < ManagedNFTsLength; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
-               ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1]; 
+               ManagedNFTs[i] = ManagedNFTs[ManagedNFTsLength - 1];
                ManagedNFTs.pop();
                break;
            } 
        }
        // By this point the NFT should have been found and removed from ManagedNFTs 
        require(true, "unable to find released NFT in ManagedNFTs"); 

        emit ReleaseManagedNFT(nftContract, to); 
```    