## 1 - Althea Liquid Infrastructure: OVERVIEW

Imagine diving into a revolutionary world where the boundaries between the physical and digital realms blur, giving rise to an exhilarating fusion of real-world assets and blockchain technology. Welcome to Liquid Infrastructure, a cutting-edge protocol that's set to redefine how we invest in, manage, and profit from real-world assets like never before!

At the heart of this groundbreaking venture is the Althea-L1 blockchain, a marvel of modern technology built on the Cosmos SDK with an Ethereum Virtual Machine compatibility layer. It's not just any blockchain; it's a powerhouse designed to support the ambitious goals of Liquid Infrastructure, enabling seamless interaction between traditional assets and the digital universe.
Picture this: everyday objects and infrastructure around you, from the router that connects you to the world, to the vending machine at your office, the solar panels powering homes, and even the electric car chargers dotting the cityscape, all tokenized as unique, digital assets on the blockchain. This isn't science fiction; it's the promise of Liquid Infrastructure.

Through the magic of LiquidInfrastructureNFT contracts, these real-world assets are transformed into Non-Fungible Tokens (NFTs), each with its unique digital identity and value, securely recorded on the blockchain. But the innovation doesn't stop there. The LiquidInfrastructureERC20 token enters the scene as a digital maestro, orchestrating the aggregation and distribution of revenue generated by these assets, directly to you, the token holder, in a fair and proportional manner.

Imagine the flexibility and opportunities at your fingertips. Want to invest in renewable energy infrastructure or a network of smart vending machines? Liquid Infrastructure makes it not just possible, but easy and transparent, offering automatic management and the ability to group these tokenized assets into various classes, represented by ERC20 tokens. It's an investor's dream, providing a novel way to diversify portfolios and tap into revenue streams from the physical world, all through the security and transparency of blockchain technology.

And while the technical wonders of the Althea-L1 chain, with its EVM compatibility and Cosmos SDK modules, set the stage, the spotlight shines brightly on Liquid Infrastructure. It's a protocol that's not just about investment; it's about revolutionizing how we interact with and benefit from the world around us, bringing us one step closer to a future where the line between physical and digital assets is not just blurred, but erased.

Get ready to be part of this thrilling journey with Liquid Infrastructure, where the future of investing is not just liquid; it's tangible, transparent, and teeming with possibilities. Welcome to the new era of tokenized real-world assets, where your next investment could be as close as the nearest electric car charger or as vast as a field of solar panels. The future is here, and it's incredibly exciting!

## 2 - Althea Liquid Infrastructure: key Features

1 - **Liquid Infrastructure as a Protocol:** It's a system that enables the conversion of real-world assets into digital tokens on a blockchain. This process is known as tokenization. The goal is to allow these tokenized assets to generate revenue, which is then recorded and managed on the blockchain.

2 - **Deployment on Althea-L1 Blockchain:** The protocol is intended to be launched on the Althea-L1 blockchain, a blockchain platform built using the Cosmos SDK (Software Development Kit) and includes an EVM (Ethereum Virtual Machine) compatibility layer. This means it can run applications designed for Ethereum, benefiting from Cosmos SDK's features while being compatible with Ethereum's tools and assets.

3 - **Tokenized Real World Assets:** The protocol involves creating digital tokens that represent real-world assets. These are managed through smart contracts on the blockchain, specifically through two types of tokens:

- LiquidInfrastructureNFT (Non-Fungible Token) contracts: These are unique digital tokens representing individual real-world assets, such as routers, vending machines, renewable energy installations, or electric car chargers. NFTs are unique, meaning each token represents a specific asset.

- LiquidInfrastructureERC20 tokens: These are fungible tokens used to aggregate and distribute the revenue generated by the NFT-represented assets. Fungible tokens are interchangeable, meaning each unit is the same as every other unit.

4 - **Flexibility and Management of Assets:** The NFTs in this protocol are designed to be versatile, capable of representing various types of physical assets. This flexibility allows for automatic management and grouping of these tokenized assets into categories, which can then be represented collectively by ERC20 tokens. This structure facilitates the investment in and revenue distribution from different classes of real-world assets through blockchain technology.

5 - **Exclusion of Althea-L1 Chain's Specifics:** The description specifies that the functionalities provided by the Althea-L1 chain, especially those related to the Cosmos SDK modules (e.g., x/bank, x/microtx), are beyond the scope of the audit or discussion. This implies that the focus is on the Liquid Infrastructure protocol itself, not the underlying blockchain's specific technical features.

In essence, Liquid Infrastructure aims to bridge the gap between physical assets and digital finance, offering a new way to invest in and manage revenue from real-world assets through blockchain technology.

## 3 - Althea Liquid Infrastructure: Contracts Overview

###### Explanation for LiquidInfrastructureERC20.sol

`LiquidInfrastructureERC20` extends several OpenZeppelin contracts to create an ERC20 token with additional features. The token represents an investment in real-world assets, specifically infrastructure involved in an Althea pay-per-forward network, and entitles holders to revenue generated by these assets.

1. **Revenue Distribution**: The contract collects revenue from managed `LiquidInfrastructureNFTs` and distributes it to token holders. The distribution is locked during a minimum period (`MinDistributionPeriod`) to prevent frequent changes in token supply that could disrupt the distribution process.

2. **Holder Allowlist**: Only approved addresses can hold the token. The owner can add or remove addresses from the allowlist.

3. **Minting and Burning Restrictions**: Minting and burning of tokens are restricted during the distribution period and can only occur after the minimum distribution period has passed.

4. **Non-Reentrancy**: The contract uses the `ReentrancyGuard` modifier to prevent reentrancy attacks during critical functions like `distribute` and `withdrawFromManagedNFTs`.

5. **Managed NFTs**: The contract interacts with `LiquidInfrastructureNFT` contracts, which are NFTs representing ownership of infrastructure assets. The contract can add or release managed NFTs.

6. **Event Logging**: The contract emits events for key actions, aiding in transparency and off-chain tracking.

7. **Token Transfer Restrictions**: The `_beforeTokenTransfer` hook enforces that transfers are not allowed during distribution and that the receiver is an approved holder.

8. **Withdrawals**: The contract can withdraw funds from managed NFTs, which are then used for distribution to token holders.

9. **Versioning**: The contract includes a `Version` constant to track contract updates.

10. **Distributable ERC20s**: The contract owner can set which ERC20 tokens are distributable to holders.

###### Explanation for LiquidInfrastructureNFT.sol

`LiquidInfrastructureNFT` inherits from `ERC721` and a custom contract `OwnableApprovableERC721`. This contract is designed to interact with the Althea network's x/microtx module, which is part of a Cosmos blockchain setup. The contract is intended to represent a Liquid Infrastructure Account, which is an account on the Cosmos Bank module that is connected to the Ethereum Virtual Machine (EVM) through Althea's x/microtx module.

***Here's a breakdown of the contract's functionality and potential security considerations:***

1. **Contract Versioning**: The contract includes a `Version` constant to indicate the version of the contract, which is a good practice for managing updates and compatibility.

2. **Single Token NFT**: The contract is designed to mint a single NFT with a token ID of `1`, which represents the Liquid Infrastructure Account. This NFT is minted to the `msg.sender` upon contract deployment.

3. **Threshold Management**: The contract allows the owner or an approved entity to set thresholds for ERC20 token balances using the `setThresholds` function. These thresholds determine how much of each token should be left in the Liquid Account's x/bank account, with excess amounts being transferred to this contract. The `getThresholds` function allows anyone to view the current thresholds.

4. **Withdrawal Functions**: The contract includes functions to withdraw ERC20 token balances (`withdrawBalances` and `withdrawBalancesTo`) to the owner of the NFT. These functions are access-controlled and emit a `SuccessfulWithdrawal` event upon successful transfer.

5. **Account Recovery**: The `recoverAccount` function allows the owner to initiate a recovery process for the Liquid Infrastructure Account. This emits a `TryRecover` event, which should be detected by the x/microtx module to transfer all tokens from the x/bank account to this contract.

###### Explanation for OwnableApprovableERC721.sol

`OwnableApprovableERC721` is an abstract contract that extends the functionality of an ERC721 token by adding two modifiers: `onlyOwner` and `onlyOwnerOrApproved`. This contract is designed to be used with tokens that have specific IDs.

***Here's a breakdown of the code:***

1. The contract inherits from `Context` (from OpenZeppelin) and `ERC721` (the standard interface for non-fungible tokens).

2. The `onlyOwner` modifier checks if the caller (`_msgSender()`, which is a function from `Context` that returns the address of the entity performing the call) is the owner of the token with the specified `tokenId`. If the caller is not the owner, the transaction is reverted with an error message.

3. The `onlyOwnerOrApproved` modifier checks if the caller is either the owner of the token, has been approved for all the owner's tokens (`isApprovedForAll`), or has been approved for the specific token (`getApproved`). This is done by calling the `_isApprovedOrOwner` internal function from the ERC721 contract. If the caller does not meet these conditions, the transaction is reverted with an error message.

## 4 - Althea Liquid Infrastructure: Potential Security Flaws

###### LiquidInfrastructureERC20.sol

***Potential security flaws and considerations:***

- **Gas Limitations**: The `distributeToAllHolders` function attempts to distribute rewards to all holders in a single transaction, which could run out of gas if there are too many holders. This is mitigated by the `distribute` function, which allows for partial distributions.

- **Centralization Risks**: The contract is `Ownable`, giving the owner significant control over the contract's critical functions, such as approving holders, minting tokens, and managing NFTs. This centralization could be a point of failure or abuse.

- **Locking Mechanism**: The contract uses a boolean `LockedForDistribution` to lock certain functions during distribution. It's crucial that this locking mechanism is robust and cannot be bypassed or lead to a permanent lock.

- **Division by Zero**: In `_beginDistribution`, the contract calculates entitlement per token by dividing the balance of each ERC20 by the total supply. If the total supply is zero, this could cause a division by zero error. However, the contract checks for a zero total supply in `_isPastMinDistributionPeriod`, which should prevent this issue.

- **Array Management**: The contract uses arrays to manage holders and managed NFTs. The removal of elements from these arrays is done by replacing the element to be removed with the last element and then popping the last element. This is an efficient way to remove elements but requires careful management to ensure consistency.

- **ERC20 Approval**: The contract does not appear to handle ERC20 token approvals for the managed NFTs, assuming that the necessary approvals are already set. This could be a point of failure if the contract does not have the required approvals to transfer ERC20 tokens during distributions.

- **Error Handling**: The contract emits events but does not always revert when an action is unsuccessful. For example, in the `distribute` function, if the `transfer` call fails, it still emits the `Distribution` event with the attempted entitlement, which could be misleading.

Overall, the contract introduces several mechanisms to manage revenue distribution and token holder rights. While it uses OpenZeppelin's secure base contracts and non-reentrancy patterns, the complexity of the distribution logic and the centralization of control warrant a thorough audit to ensure security and correctness.

###### LiquidInfrastructureNFT.sol

**Potential Security Flaws and Considerations**:

- **Event Reliance**: The contract relies on events (`TryRecover` and `SuccessfulRecovery`) to communicate with the x/microtx module. It's important to ensure that the off-chain or cross-chain components listening for these events are secure and reliable.

- **Array Length Check**: In `setThresholds`, there is a check to ensure that the lengths of the `newErc20s` and `newAmounts` arrays are equal. This is a good practice to prevent mismatches in related data.

- **ERC20 Transfer Check**: The contract checks the return value of the ERC20 `transfer` function, which is important to ensure that transfers are successful.

- **Deletion of Thresholds**: The contract deletes the existing thresholds before setting new ones. This is a safe approach to prevent stale or outdated data.

- **Gas Optimization**: The contract could be optimized for gas usage. For example, the `setThresholds` function could be optimized by avoiding the deletion and repopulation of the entire arrays if only a few elements need to be changed.

- **Error Messages**: The contract uses descriptive revert messages, which is a good practice for debugging and understanding why a transaction failed.

- **Cosmos-EVM Interaction**: The contract is designed to interact with a Cosmos module. It's important to ensure that the security and correctness of these interactions are thoroughly reviewed, especially since cross-chain communication can introduce additional complexities.

Overall, the contract appears to be well-structured with attention to access control and interaction with the Cosmos network. However, a thorough audit would require reviewing the off-chain components that interact with this contract's events. Additionally, testing and code review should be conducted to ensure that all functionalities work as intended and that there are no vulnerabilities in the cross-chain communication logic.

## 5 -  Althea Liquid Infrastructure: Conclusion


In conclusion, this audit unveils the groundbreaking potential of Althea Liquid Infrastructure, where the realms of physical assets and blockchain technology converge to redefine investment paradigms. With an ambitious vision and a meticulously crafted protocol, Liquid Infrastructure promises a future where real-world assets seamlessly interact with the digital realm, offering unparalleled transparency, flexibility, and opportunity.

While the audited smart contracts showcase robust mechanisms for asset tokenization, revenue distribution, and access control, they also highlight critical areas for further scrutiny and enhancement. By addressing potential security flaws and optimizing gas usage, the project can bolster confidence in its infrastructure and pave the way for a seamless integration of physical assets into the blockchain ecosystem.

As we stand on the cusp of a new era in decentralized finance, Althea Liquid Infrastructure beckons investors and enthusiasts alike to embark on an exhilarating journey where innovation meets possibility. With diligent oversight and continuous refinement, the project is poised to realize its vision of a future where the boundaries between the physical and digital worlds dissolve, ushering in an era of unprecedented opportunity and prosperity.



### Time spent:
20 hours