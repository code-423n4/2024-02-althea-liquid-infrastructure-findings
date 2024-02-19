## LiquidInfrastructureERC20 expects to receive ERC721 NFT tokens but doesn't implement `onERC721Received`

## Links
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L395

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L414-L417

## Vulnarability Details

The `LiquidInfrastructureERC20` will receive NFTs, however if the method used for such transfer is `safeTransferFrom`, the transfer call will be reverted as the `LiquidInfrastructureERC20` doesn't implement the onERC721Received function. 

## Impact

When transferring NFTs to the contract with `safeTransferFrom` or `safeMint` functions, the call will revert. 

## Mitigation step

In order to mitigate this issue add the onERC721Received function to the contract.

```
    function onERC721Received(address, address, uint256, bytes memory) public pure returns (bytes4) {
        // Handle the token transfer here
        return this.onERC721Received.selector;
    }
```
