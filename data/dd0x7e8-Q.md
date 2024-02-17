## Incompatible with USDT contract

Certain stablecoin contracts, such as [USDT](https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code), deviate from the OpenZeppelin ERC20 standard by not returning a boolean value upon completing a transfer. This deviation can lead to consistent reversion of transfer transactions.

Within the `distribute()` function, the `distributableERC20s[j]` provided may correspond to one of these non-compliant tokens, potentially triggering the issue described above.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L221C1-L226C22

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L179C1-L179C76

To mitigate the potential issue, it is advisable to utilize OpenZeppelin's SafeERC20 wrapper.

## Employing `transferFrom()` as opposed to `safeTransferFrom()` for NFT transactions may result in the irreversible loss of the NFTs.

The EIP-721 standard says the following about `transferFrom()`:

```solidity
    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```

https://github.com/ethereum/EIPs/blob/78e2c297611f5e92b6a5112819ab71f74041ff25/EIPS/eip-721.md?plain=1#L103-L113 Code must use the `safeTransferFrom()` flavor if it hasn't otherwise verified that the receiving address can handle it.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L418C1-L418C62

```solidity
        nft.transferFrom(address(this), to, nft.AccountId());
```

Consider using the `safeTransferFrom()` to transfer the ERC721 token. If `to` refers to a smart contract, it must implement {IERC721Receiver-onERC721Received}, which is called upon a safe transfer.

## Redundant Code
The `require` statement is redundant and should be removed to save gas.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431C1-L431C69

```solidity
        require(true, "unable to find released NFT in ManagedNFTs");
```

Recommend removing the code to save gas.


