### LiquidInfrastructureERC20.sol doesn't have the ability to call recoverAccount
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L27-L31
> Occassionally devices and wallets can be lost, in which case the owner of this contract can call recoverAccount() to begin a recovery process which will finish after the transaction completes. Asynchronously the x/microtx module will ignore thresholds and transfer all of the Liquid Account's balances to this NFT, which may be withdrawn normally with withdrawBalances().

Currently there is no way to call `recoverAccount()`, in case of emergency, if NFT belongs to `LiquidInfrastructureERC20.sol` contract.

### Some ERC20 tokens may become stuck inside LiquidInfrastructureERC20.sol
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L220
```solidity
>>              for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }
```
When `withdrawFromManagedNFTs` is called all tokens from `thresholdsErc20s` array
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L39
are transferred to the `LiquidInfrastructureERC20.sol` address
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L376-L377
```solidity
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

>>          (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
>>          withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
```
However, there is no check if tokens from `withdrawERC20s` are in `distributableERC20s` array. As a result some tokens may be undistributed. 

### No duplicates check in addManagedNFT
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394-L403
```solidity
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
>>      ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```
When the owner adds a new liquidity NFT there is no check if it's already present in `ManagedNFTs` array. If it is later released, only one duplicate address is removed.
```solidity
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
>>          if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
>>              break;
            }
        }
```
This will cause a temporarily DOS on `withdrawFromManagedNFTs` call since liquidity ERC20 doesn't own the NFT.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L377

### Updating distributableERC20s array during distribution phase may break the entitlement accounting
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L205
When `distribute` function is called, the contract precalculates entitlement values for tokens stored in `distributableERC20s` array
```solidity
        // Calculate the entitlement per token held
        uint256 supply = this.totalSupply();
        for (uint i = 0; i < distributableERC20s.length; i++) {
            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );
            uint256 entitlement = balance / supply;
>>          erc20EntitlementPerUnit.push(entitlement);
        }
```
Note that the entitlements are pushed into the array in the same order that tokens are stored in `distributableERC20s`.  If the contract owner attempts to change distributable tokens with `setDistributableERC20s` during the distribution phase
```solidity
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```
, and new tokens are stored in a different order, wrong amounts will be distributed.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L220
```solidity
                for (uint j = 0; j < distributableERC20s.length; j++) {
>>                  IERC20 toDistribute = IERC20(distributableERC20s[j]);
>>                  uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }
```
Consider adding this check to `setDistributableERC20s`
```solidity
require(!LockedForDistribution,"distribution is in process");
```

### Blacklisted approved holder may temporarily stop the distribution
Some tokens (e.g. USDT, USDC) have a blackist which prevents transferring tokens from or to certain accounts. If one of the approved holders is blacklisted, distribution will revert.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224

### Some tokens (e.g. USDT) don't return bool value on transfer
NOT MENTIONED IN https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md#m-03-unsafe-use-of-transfertransferfrom-with-ierc20

As a result some receipts won't be included in `Distribution` event

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224-L226
```solidity
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
>>                  if (toDistribute.transfer(recipient, entitlement)) {
                        receipts[j] = entitlement;
                    }
                }

>>              emit Distribution(recipient, distributableERC20s, receipts);
```

### Double check in distribute function
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L200
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L258-L261

Since `distribute` function is the only entry point for `_beginDistribution`, and we've already checked `!LockedForDistribution`, we can discard the same check in `_beginDistribution`.

### constructor doesn't check if the contract is managed NFT owner
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L459
```solidity
    constructor(
        string memory _name,
        string memory _symbol,
>>      address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
        ManagedNFTs = _managedNFTs;
        LastDistribution = block.number;

        for (uint i = 0; i < _approvedHolders.length; i++) {
            HolderAllowlist[_approvedHolders[i]] = true;
        }

        MinDistributionPeriod = _minDistributionPeriod;

        distributableERC20s = _distributableErc20s;

        emit Deployed();
    }
```
Consider adding the same check as in `addManagedNFT`

```solidity
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
>>      require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```