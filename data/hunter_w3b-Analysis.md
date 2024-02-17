# Analysis - Althea Liquid Infrastructure Contest

![Althea-Liquid-Infrastructure-Protocol](https://code4rena.com/_next/image?url=https%3A%2F%2Fstorage.googleapis.com%2Fcdn-c4-uploads-v0%2Fuploads%2Fy3r4wwbxrwB.0&w=256&q=75)

## Description overview of Althea Liquid Infrastructure Contest

The Althea Liquid Infrastructure is a protocol designed to facilitate the `tokenization` and `investment` in real-world assets, enabling revenue accrual `on-chain`. It consists of tokenized real-world assets represented `on-chain` by deployed `LiquidInfrastructureNFT` contracts and the `LiquidInfrastructureERC20` token. The protocol allows for the aggregation and distribution of revenue proportionally to holders of the ERC20 token.

Key features and aspects of Althea Liquid Infrastructure include:

1. **Tokenized Real-World Assets**: The protocol enables the tokenization of real-world assets such as devices (e.g., routers participating in Althea's pay-per-forward billing protocol), vending machines, renewable energy infrastructure, or electric car chargers.

2. **LiquidInfrastructureNFTs**: These NFTs represent the revenue generated by a microtransaction account. They can be utilized in DeFi to transfer the account, aggregate the revenue, and manage the device wallet itself. LiquidInfrastructureNFTs serve as a new DeFi primitive, connecting on-chain assets to the real world without the need for a custodian.

3. **Revenue Aggregation and Distribution**: The `LiquidInfrastructureERC20` token aggregates and distributes revenue generated by tokenized assets to its holders. Holders of the `ERC20` token are entitled to revenue from the network represented by the token.

Overall, Althea Liquid Infrastructure aims to bridge the gap between DeFi and the real world by offering a native, non-custodial solution for tokenizing and managing real-world assets on-chain, while enabling revenue accrual and distribution to token holders.

**The key contracts of Althea Liquid Infrastructure protocol for this Audit are**:

`LiquidInfrastructureERC20.sol`, `LiquidInfrastructureNFT.sol`, and `OwnableApprovableERC721.sol`. These contracts form the backbone of the protocol, enabling the aggregation and distribution of revenue from managed Liquid Infrastructure NFTs to token holders, facilitating investment in real-world assets represented on-chain, and providing enhanced ownership and control functionalities through ERC721 NFTs. Each contract plays a critical role in the protocol's functionality, security, and interoperability, making them essential components for review and analysis during the audit process.

## System Overview

### High-level System Overview

![Althea](https://private-user-images.githubusercontent.com/70327041/305649653-aa2bcfe3-5890-42cc-b237-795cf0f8d266.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDgxODA0NDYsIm5iZiI6MTcwODE4MDE0NiwicGF0aCI6Ii83MDMyNzA0MS8zMDU2NDk2NTMtYWEyYmNmZTMtNTg5MC00MmNjLWIyMzctNzk1Y2YwZjhkMjY2LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAyMTclMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMjE3VDE0MjkwNlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWNiYmFkYmE4ODU0MDM5Y2NlODgyMzlmYmQzYTNmNjI3NDY2YmUyMGUxNDRhMGZmMWZiYThiODg0ZTU1NmI4ODQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.TqzPMl3SJP5xQwAUuaxGs33hSPDQ4c6gYVWTFaDYIrA)

### Scope

- **LiquidInfrastructureERC20.sol**: This contract, called `Liquid Infrastructure ERC20`, enables the aggregation and distribution of revenue from managed Liquid `Infrastructure NFTs` to token holders, facilitating investment in real-world assets represented on-chain.It manages an ERC20 token that represents ownership of `liquid infrastructure assets` represented by `LiquidInfrastructureNFT` contracts. Revenue generated by the `LiquidInfrastructureNFTs` is collected by this contract and periodically distributed to ERC20 token holders. It tracks a list of approved ERC20 token holders who are eligible to receive distributions. Distributions are locked between minimum time periods to prevent gaming the system. The contract periodically withdraws funds from the `LiquidInfrastructureNFTs` and calculates each token holder's share based on their balance. Token holders passively earn revenue from the real-world infrastructure assets as the underlying NFTs generate payments. The contract owner can manage the NFT assets, adding/removing them over time to control the revenue streams. Burns and mints of the ERC20 token are also locked during distributions to maintain accurate balances.

- **LiquidInfrastructureNFT.sol**: This contract, called `Liquid Infrastructure NFT`, is an ERC721 NFT contract designed to control a Liquid Infrastructure Account. Each NFT represents a single "`liquid infrastructure account`" on Cosmos that receives payments. It tracks ERC20 "thresholds" that determine how much of each token can stay in the Cosmos account. The Cosmos `x/microtx` module monitors thresholds and sends excess funds to the NFT as ERC20s. The owner can withdraw accumulated ERC20 balances from the NFT to their wallet. If the Cosmos account is lost, the owner can trigger a recovery to migrate all funds to the NFT. This enables passive income from real-world infrastructure paid in the native Cosmos coins. The ERC721 NFT represents tokenized ownership of the Cosmos account and accumulated funds. By coupling the NFT to the Cosmos account, it enables withdrawal of both Cosmos coins and assets bridged to ERC20. This contracts provides interoperability between Cosmos liquidity providers and Ethereum DeFi users/protocols.

- **OwnableApprovableERC721.sol**: This abstract contract, called `OwnableApprovableERC721`, extends ERC721 functionality and provides modifiers `onlyOwner(id)` and `onlyOwnerOrApproved(id)` to restrict access to specific functions based on ownership or approval status of a token with the given ID.

### Chains supported

Only Althea-L1 is supported.

### Roles

1.  **LiquidInfrastructureERC20.sol**:

    - **Owner**:

      - The owner has administrative control over the contract.
      - Can add or remove approved holders.
      - Can start and manage distribution of rewards.
      - Can mint new tokens and burn existing ones.
      - Manages the list of managed LiquidInfrastructureNFT contracts.
      - Can modify the list of ERC20s eligible for distribution.

    - **Token Holders**:

      - Holders of the ERC20 tokens.
      - Can receive rewards distributed from managed LiquidInfrastructureNFTs.
      - Need to be approved by the owner to hold tokens.

2.  **LiquidInfrastructureNFT.sol**:

    - **Owner**:

      - The owner has administrative control over the contract.
      - Can set and update threshold values for ERC20 tokens.
      - Can withdraw ERC20 balances held by the contract.
      - Can initiate the account recovery process.
      - Manages the ownership and control of the Liquid Infrastructure NFT.

    - **Token Holders**:

      - Token holders represent the owner of the Liquid Infrastructure Account controlled by the NFT.
      - Holders of the NFT have control over setting threshold values and withdrawing ERC20 balances.

### Invariants Generated

#### Invariants for the LiquidInfrastructureERC20 Contract:

1. **Minimum Distribution Period Enforcement**:

   - Ensure that the minimum distribution period is enforced before allowing minting or burning of tokens.
   - **Failure Scenario**: If minting or burning is allowed without respecting the minimum distribution period, it could lead to unfair distribution of rewards or manipulation of the token supply.

2. **Lock During Distributions**:

   - Prevent transfers, mints, and burns of the token while a distribution is in progress.
   - **Failure Scenario**: Allowing transfers, mints, or burns during a distribution could lead to inconsistencies in reward distribution or manipulation of token balances.

3. **Holder Approval**:

   - Ensure that only approved holders can receive tokens.
   - **Failure Scenario**: If tokens are sent to unapproved holders, it could result in unauthorized token distribution or loss of tokens.

4. **Distribution Completion**:

   - Ensure that a distribution is completed before allowing other operations.
   - **Failure Scenario**: Failing to complete a distribution before starting another one could lead to incorrect distribution of rewards or locking up of tokens.

5. **Distribution Start Period Check**:

   - Verify that distributions can only begin once every minimum distribution period.
   - **Failure Scenario**: Starting distributions too frequently could lead to excessive gas costs, congestion, or unfair reward distribution.

6. **Balance and Holder Management**:
   - Ensure that holders are added and removed correctly based on their token balances.
   - **Failure Scenario**: Incorrect management of holder lists could lead to misrepresentation of token holders or incorrect distribution of rewards.

#### Invariants for the LiquidInfrastructureNFT Contract:

1. **Threshold Consistency**:

   - Ensure that the number of threshold values matches the number of associated ERC20 addresses.
   - **Failure Scenario**: If the lengths of `newErc20s` and `newAmounts` arrays passed to `setThresholds` differ, it could lead to inconsistency in controlling the operating balances of the liquid account or unexpected behavior in the withdrawal process.

2. **Access Control for Threshold Modification**:

   - Ensure that only the owner or an approved entity can modify the threshold values.
   - **Failure Scenario**: Unauthorized modification of threshold values could result in manipulation of the liquid account's operating balances or unauthorized access to its funds.

3. **Correct Withdrawal Execution**:

   - Ensure that ERC20 balances are withdrawn correctly to the specified destination address.
   - **Failure Scenario**: If the withdrawal function fails to transfer ERC20 balances or sends them to an incorrect destination, it could result in loss of funds or incorrect distribution of assets.

4. **Recovery Process Integrity**:

   - Ensure that the recovery process is initiated only by the owner of the NFT.
   - **Failure Scenario**: If the recovery process is triggered by an unauthorized entity, it could lead to unauthorized access to the liquid account's funds or disruption of the recovery process.

5. **Consistent Event Emission**:
   - Ensure that events are emitted consistently upon successful actions like withdrawal or recovery.
   - **Failure Scenario**: Inconsistent event emission could make it difficult to track the state changes of the contract or signal the completion of critical processes.

## Approach Taken-in Evaluating The Althea Liquid Infrastructure Protocol

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Althea Liquid Infrastructure Protocol.

    I start with the following contracts, which play crucial roles in the Althea Liquid Infrastructure protocol:

    **Main Contracts I Looked At**

    I start with the following contracts, which play crucial roles in the Althea Liquid Infrastructure protocol:

                LiquidInfrastructureERC20.sol
                LiquidInfrastructureNFT.sol
                OwnableApprovableERC721.sol

    I started my analysis by examining the intricate structure and functionalities of the `Althea Liquid Infrastructure` protocol, which aims to bridge the gap between traditional finance and decentralized finance (DeFi) by enabling the tokenization and investment in real-world assets on-chain. This protocol facilitates revenue accrual on-chain through the tokenization of real-world assets, which are represented by deployed `LiquidInfrastructureNFT` contracts and the `LiquidInfrastructureERC20` token. Through a comprehensive examination of these key contracts and their interactions, I sought to understand how the protocol manages revenue distribution, token ownership, and account recovery processes. Additionally, I investigated the role of access control mechanisms and the integrity of event emission to assess the protocol's security and reliability.

2.  **Documentation Review**:

    Then went to Review [This articles](https://medium.com/althea-mesh/introducing-liquid-infrastructure-a-new-defi-primitive-f4e2df14088b) and [this](https://medium.com/althea-mesh) for a more detailed and technical explanation of the Althea Liquid Infrastructure protocol.

3.  **Compiling code and running provided tests**:

4.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

### Protocol Structure

## Codebase Quality

Overall, I consider the quality of the Althea Liquid Infrastructure protocol codebase to be Good. The code appears to be mature and well-developed. We have noticed the implementation of various standards adhere to appropriately. Details are explained below:

| Codebase Quality Categories              | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code Maintainability and Reliability** | The Althea Liquid Infrastructure Protocol contracts demonstrates good maintainability through modular structure, consistent naming, and efficient use of libraries. It also prioritizes reliability by handling errors, conducting security checks. However, for enhanced reliability, consider professional security audits and documentation improvements. Regular updates are crucial in the evolving space.                                                                                                                                                                                                                                                                                                            |
| **Code Comments**                        | During the audit of the Althea Liquid Infrastructure contracts codebase, I found that some areas lacked sufficient internal documentation to enable independent comprehension. While comments provided high-level context for certain constructs, lower-level logic flows and variables were not fully explained within the code itself. Following best practices like NatSpec could strengthen self-documentation. As an auditor without additional context, it was challenging to analyze code sections without external reference docs. To further understand implementation intent for those complex parts, referencing supplemental documentation was necessary.                                                      |
| **Documentation**                        | There is no good documentation and Video of the Althea Liquid Infrastructure project to providing a solid overview of how Althea Liquid Infrastructure protocol is structured and how its various aspects function. However, also we have noticed that there is room for additional details, such as `diagrams`, to gain a deeper understanding of how different contracts interact and the functions they implement. With considerable enthusiasm. We are confident that these diagrams will bring significant value to the protocol as they can be seamlessly integrated into the existing documentation, enriching it and providing a more comprehensive and detailed understanding for users, developers and auditors. |
| **Testing**                              | The audit scope of the contracts to be audited is 95.09% this is Good.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| **Code Structure and Formatting**        | The codebase contracts are well-structured and formatted. but some order of functions does not follow the Solidity Style Guide According to the Solidity Style Guide, functions should be grouped according to their visibility and ordered: constructor, receive, fallback, external, public, internal, private. Within a grouping, place the view and pure functions last.                                                                                                                                                                                                                                                                                                                                               |
| **Error**                                | Use custom errors, custom errors are available from solidity version 0.8.4. Custom errors are more easily processed in try-catch blocks, and are easier to re-use and maintain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **Imports**                              | In Althea Liquid Infrastructure protocol the contract's interface should be imported first, followed by each of the interfaces it uses, followed by all other files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

## Systemic & Centralization Risks

The analysis provided highlights several significant systemic and centralization risks present in the Althea Liquid Infrastructure protocol. These risks encompass concentration risk in `LiquidInfrastructureERC20.sol`, `LiquidInfrastructureNFT.sol` risk and more, third-party dependency risk, and centralization risks arising.

Here's an analysis of potential systemic and centralization risks in the contracts:

### Systemic Risks:

1. **LiquidInfrastructureERC20.sol**

   - **Locking of Transfers**: The contract locks transfers, mints, and burns during distribution periods to ensure fair distribution of rewards. This could inconvenience users who wish to trade tokens during these periods.

   - **Out of Gas Errors**: Distributing rewards to a large number of holders may exceed the block gas limit, leading to out of gas errors and incomplete distributions.

   - **Oracle risk**: Revenue amounts bridged from Cosmos rely on accurate price feeds.

2. **LiquidInfrastructureNFT.sol**:

   - **Threshold Management**: The contract manages thresholds for ERC20 tokens, determining how much of each token should be left in the Liquid Account's x/bank account. Incorrectly setting or updating these thresholds could result in improper handling of token balances.

   - **Account Recovery Process**: The account recovery process relies on external interactions with the x/microtx module. Delays or failures in this process could affect the recovery of ERC20 token balances.

### Centralization Risks:

1. **LiquidInfrastructureERC20.sol**

   - **Owner Control**: The owner has significant control over contract functionalities, including `adding/removing` approved holders, starting distributions, and managing the list of managed NFTs and ERC20s. This centralized control could lead to governance issues or unfair treatment of token holders.

   - **Approved Holder System**: The contract relies on an approved holder system where the owner approves specific addresses to hold the ERC20 tokens. This centralization of approval authority could be perceived as centralized control over token ownership.

   - **Managed NFTs**: The contract manages a collection of LiquidInfrastructureNFTs, with the owner having the ability to add or release these NFTs. This centralization of control over managed NFTs could lead to potential manipulation or favoritism.

2. **LiquidInfrastructureNFT.sol**:

   - **Owner Control**: The owner has significant control over contract functionalities, including setting thresholds, initiating account recovery, and withdrawing ERC20 balances. This centralized control could lead to governance issues or unfair treatment of token holders.

   - **Threshold Setting**: While token holders can set and update threshold values, the initial thresholds are set by the contract owner. This centralization of control over initial threshold values could be perceived as centralized control over the handling of ERC20 token balances.

   - **Account Recovery**: Only the owner can initiate the account recovery process. This centralization of control over account recovery could lead to delays or issues if the owner is unavailable or unresponsive.

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Althea Liquid Infrastructure protocol.**

## Suggestions

### Suggestions for the Althea Liquid Infrastructure Protocol

1. **Continuous Monitoring and Auditing**: Implement continuous monitoring and auditing processes to detect and address potential vulnerabilities, bugs, and security issues proactively. Utilize automated security tools and services to perform regular security scans and audits of the codebase.

2. **Comprehensive Documentation and Testing**: Enhance the documentation of the codebase with detailed explanations of contract functionalities, usage examples, and security considerations. Invest in comprehensive unit tests, integration tests, and fuzz testing to validate the correctness, robustness, and security of the protocol.

### What’s unique?

The unique aspects of the Althea Liquid Infrastructure protocol include:

1. **Tokenization of Real-World Assets**: The protocol enables the tokenization of real-world assets such as devices, vending machines, and renewable energy infrastructure, allowing them to be represented and traded on-chain.

2. **LiquidInfrastructureNFTs**: These NFTs represent ownership of microtransaction accounts linked to real-world infrastructure, providing a novel mechanism for managing and monetizing infrastructure assets in DeFi.

3. **Revenue Distribution Mechanism**: The protocol aggregates and distributes revenue generated by tokenized assets to holders of the `LiquidInfrastructureERC20` token, enabling passive income opportunities for token holders.

### What ideas can be incorporated?

1. **Integration with Oracles**: Incorporate decentralized oracles to provide real-time data feeds on asset performance, revenue generation, and market conditions, enhancing the transparency and reliability of the protocol.

2. **Governance Mechanisms**: Implement decentralized governance mechanisms to enable token holders to participate in protocol governance, including voting on proposals, parameter adjustments, and upgrades.

3. **Liquidity Mining and Incentives**: Introduce liquidity mining programs and incentives to bootstrap liquidity, attract users, and incentivize participation in the protocol ecosystem, driving adoption and growth.

## Conclusion

The Althea Liquid Infrastructure protocol presents an innovative solution for tokenizing and managing real-world assets on-chain, while enabling revenue accrual and distribution to token holders. By leveraging tokenized assets represented by `LiquidInfrastructureNFTs` and the `LiquidInfrastructureERC20` token, the protocol bridges the gap between DeFi and the real world, offering a non-custodial solution for investment in infrastructure assets.

Through a comprehensive review of the protocol's codebase and functionalities, several key insights and recommendations have been identified to enhance the security, efficiency, and usability of the protocol. Recommendations include implementing enhanced security measures, implementing continuous monitoring and auditing processes, and enhancing documentation and testing practices.

Furthermore, the protocol's unique features, such as tokenization of real-world assets, `LiquidInfrastructureNFTs`, and revenue distribution mechanisms, present opportunities for incorporating additional features such as integration with oracles, governance mechanisms, liquidity mining incentives, and cross-chain compatibility.

Overall, the Althea Liquid Infrastructure protocol demonstrates significant potential to transform the landscape of asset tokenization and management, offering a robust and decentralized solution for users to participate in the ownership and revenue generation of real-world assets through DeFi. With continued development, innovation, and community engagement, the protocol is poised to achieve widespread adoption and contribute to the evolution of decentralized finance.


### Time spent:
30 hours