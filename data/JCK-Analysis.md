

## Table of Contents

| Number | Particulars | 
|--------|-------------|
| 1 | Overview  |
| 2 | Approach taken in evaluating the codebase |
| 3 | Centralization risks |
| 4 | Mechanism review |
| 5 | Systemic Risks |
| 6 | Technical Overview |
| 7 | Recommendation |


1. Overview

LiquidInfrastructureERC20, is an ERC-20 token contract implemented in Solidity. It serves as a mechanism for users to earn rewards from managed Liquid Infrastructure NFTs (Non-Fungible Tokens). The rewards are distributed from these NFTs to the token holders on a semi-regular basis. The contract also includes features such as minting, burning, distribution locking, and management of approved holders.
LiquidInfrastructureNFT, is an ERC-721 NFT contract implemented in Solidity. It is designed to control a Liquid Infrastructure Account, a Cosmos Bank module account connected to the EVM through Althea's x/microtx module. The contract includes functionality for managing thresholds, recovering accounts, and withdrawing ERC20 balances.
the OwnableApprovableERC721. It serves as a base contract and provides two modifiers: onlyOwner and onlyOwnerOrApproved, which are derived from ERC721's ownerOf, getApproved, and isApprovedForAll functions. The contract aims to simplify access control for contracts with manually enumerated tokens by allowing only the owner or approved addresses to execute certain functions.

2. Approach taken in evaluating the codebase:

The LiquidInfrastructureERC20 code follows a modular approach, importing various OpenZeppelin contracts to leverage their functionality, such as ERC20, Ownable, ERC721Holder, ERC20Burnable, and ReentrancyGuard. The code is well-commented, providing explanations for each function and section. The use of events (event) is employed for emitting important contract events, aiding in transparency and monitoring.
The LiquidInfrastructureNFT code is well-structured and modular, utilizing OpenZeppelin contracts and custom OwnableApprovableERC721 for access control. The contract includes clear comments for each function, explaining its purpose and usage. The ERC721 standard is extended, and the codebase follows best practices.
the OwnableApprovableERC721 It serves as a base contract and provides two modifiers: onlyOwner and onlyOwnerOrApproved, which are derived from ERC721's ownerOf, getApproved, and isApprovedForAll functions. The contract aims to simplify access control for contracts with manually enumerated tokens by allowing only the owner or approved addresses to execute certain functions.

3. Centralization risks

The ownership of LiquidInfrastructureERC20 the contract is concentrated in a single Owner, and actions like adding/removing approved holders, managing NFTs, and setting distributable ERC20s are exclusively under the control of the contract owner. The centralization of these functions may pose a risk, especially if the owner's account is compromised or if there is a lack of governance mechanisms.
OwnableApprovableERC721 is minimal in this specific contract since it serves as an abstract base contract providing access control modifiers. The centralization risk may arise if the derived contracts that use these modifiers have centralized control over critical functions.

4. Mechanism review 

in the LiquidInfrastructureERC20
Distributing rewards to token holders from managed NFTs.
Managing the whitelist of approved holders.
Minting and burning functions, subject to distribution periods.
Controlling and releasing managed NFTs.
Setting distributable ERC20s.

in the LiquidInfrastructureNFT contract  Setting and updating thresholds for controlling the operating balances of the Liquid Account.
Withdrawing ERC20 balances from the contract to the owner or a specified destination.
Initiating the recovery process for the Liquid Infrastructure Account.
Emitting events for successful withdrawals, recovery attempts, and successful recoveries.

The LiquidInfrastructureERC20 contract include two modifiers:

onlyOwner(uint256 tokenId): Allows a function to be executed only by the owner of a specific ERC721 token.
onlyOwnerOrApproved(uint256 tokenId): Allows a function to be executed by either the owner or an address approved for the specified ERC721 token.
These modifiers enhance the security and control of functions in derived contracts.

5. Systemic Risks

Potential systemic risks include vulnerabilities in external dependencies (e.g., OpenZeppelin contracts) and issues related to gas limits during large-scale distributions to numerous token holders. Regular audits and keeping dependencies up-to-date can mitigate these risks.

in the OwnableApprovableERC721 contract 
Systemic risks are minimal in this contract, as it mainly consists of modifiers and relies on ERC721 standard functions. However, potential risks may arise if derived contracts do not implement access control correctly or if there are vulnerabilities in the ERC721 standard.

6. Technical Overview
The code is written in Solidity version 0.8.12 and utilizes various OpenZeppelin contracts for standard ERC-20 functionality, access control, and security measures. The contract seems well-structured and modular, with clear separation of concerns. It uses a semi-automatic distribution mechanism to reward token holders based on their holdings. The addition and removal of managed NFTs are handled securely.

7. Recommendation:

in the LiquidInfrastructureERC20 contract Consider implementing a decentralized governance mechanism to manage critical functions, reducing centralization risks. This could involve introducing a governance token and allowing token holders to vote on key decisions. Additionally, a time-lock mechanism on certain critical functions could add an extra layer of security.
in the  LiquidInfrastructureNFT contract   Consider implementing decentralized governance mechanisms to handle critical functions such as setting thresholds and initiating the recovery process. This could involve introducing a governance token and allowing token holders to vote on these decisions. Additionally, a time-lock mechanism on certain critical functions could add an extra layer of security.
The OwnableApprovableERC721 contract is implemented in Solidity version 0.8.12 and relies on OpenZeppelin's ERC721 contract. It effectively abstracts access control concerns for ERC721 tokens and provides easy-to-use modifiers for derived contracts.


## Time 15 Hours

### Time spent:
15 hours