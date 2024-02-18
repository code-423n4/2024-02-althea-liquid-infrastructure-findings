## [L-1] `LiquidInfrastructureNFT.sol` Does Not Implement `supportsInterface` function

Contracts must properly implement the `supportsInterface` function to ensure they comply with ERC721 standards and interoperate with other contracts correctly. Implement the `supportsInterface` function to return true for `ERC721` token types, ensuring accurate reporting of supported features.

```javascript
    function supportsInterface(bytes4 _interfaceId)
            public
            view
            override(ERC721)
            returns (bool)
        {
            return super.supportsInterface(_interfaceId);
        }
```