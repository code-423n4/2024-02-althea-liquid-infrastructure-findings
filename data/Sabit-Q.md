1. Approving and Distributing to Zero Address

The approveHolder() function allows any address, including address(0), to be whitelisted as a valid holder for distributions. This can lead to tokens being sent to the zero address, locking them forever.

Reproduction:

Call approveHolder(address(0)) to add zero address to whitelist
Call distribute() function
Address(0) will be treated as valid recipient and have tokens transferred to it

The zero address is able to be whitelisted and receive token distributions, even though it does not represent a real holder. This locks sent tokens forever.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L106

Recommendation:

In approveHolder(), add require(holder != address(0)) to prevent approving zero address




2.  Burning 0 Tokens Allowed
The burnAndDistribute() function allows calling burn() with an amount of 0. 

The burn() function also lacks a require check that amount > 0.

So burning 0 tokens is not prevented or disallowed.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L331

Recommendation:

In burnAndDistribute(), add: require(amount > 0, "Cannot burn 0 tokens").


3. Distribution Event Emitted Incorrectly

The Distribution event is emitted outside the distribution loop and does not iterate over each receipt. This causes it to only emit the receipts for the last distribution recipient.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L229

Reproduction:

Call distribute() with multiple recipients
The Distribution event will only include receipts for the last recipient.

The event is emitted after the loop, so only has data for the last recipient.

Code Issue:

- The Distribution event is emitted after the distribution loop ends
- The receipts array is not iterated over to emit an event per receipt.

Recommendations:

- Move the Distribution event emit inside the distribution loop
- Include the iteration in the receipts emitted - "receipts[j]"


4. Transferring NFT to Zero Address

he releaseManagedNFT() function allows transferring a managed NFT to the zero address, which permanently locks the NFT.

Impact:

Transferring an NFT to 0x0 permanently locks it and makes it unusable.
This can lead to loss of managed NFT assets.
Reproduction:

Call releaseManagedNFT() with 0x0 as the 'to' address.
The NFT will be transferred to 0x0 and locked.
Expected:NFTs should only be allowed to be transferred to valid, non-zero addresses.

Actual:NFTs can be transferred to the zero address.

Code Issue:There is no validation on the 'to' address passed to releaseManagedNFT().

Recommendations:

Add require(to != address(0)) to prevent zero address.

Consider reverting with a custom error message.

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413-L429


