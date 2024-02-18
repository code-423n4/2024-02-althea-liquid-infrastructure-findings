


# 🛠️ Analysis - Althea Liquid Infrastructure Audit 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Security Approach of the Project | Audit approach of the Project |
|e) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|f) |Packages and Dependencies Analysis | Details about the project Packages |
|g) |Other recommendations | What is unique? How are the existing patterns used? |
|h) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/tree/main/liquid-infrastructure#compiling-the-contracts)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [Althea Liquid Infrastructure](https://github.com/althea-net/liquid-infrastructure-contracts) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/crytic/slither)| The project does not currently have a slither result, a slither control was created from initial|
|5|Test Suits|[Tests](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/tree/main/liquid-infrastructure#testing-the-contracts)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/tree/main?tab=readme-ov-file#scope)|
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/tree/main?tab=readme-ov-file#attack-ideas-where-to-look-for-bugs)||

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit

In addition, the auditor can decide on the question of where should I end the audit by examining these metrics;

Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

<img width="816" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/cd7e9336-b60e-42c6-a4c1-d58cc075d44c">





</br>
</br>

## Project - Entry Point;

1. **LiquidInfrastructureERC20**: This contract represents an ERC20 token that aggregates revenue from managed LiquidInfrastructureNFTs. It allows for the minting and burning of tokens, restricted under certain conditions related to distribution periods. It also handles the distribution of revenue to token holders and manages a list of approved holders and managed NFTs. This contract interacts with ERC20 tokens for distribution and with LiquidInfrastructureNFTs for managing infrastructure assets.

2. **LiquidInfrastructureNFT**: This contract represents a non-fungible token (NFT) that corresponds to infrastructure assets in the Althea network. It is designed to receive revenue in the form of ERC20 tokens, which can be withdrawn by the owner. This contract also allows for setting balance thresholds for automatic revenue conversion and withdrawal, and it has a recovery mechanism for lost accounts.

3. **OwnableApprovableERC721**: An abstract contract that extends ERC721 with ownership and approval checks tailored to single or specific token IDs. It provides modifiers for functions that should only be accessible by the token owner or an approved delegate. This contract is a base for the LiquidInfrastructureNFT, adding specific ownership functionality on top of the standard ERC721 features.


<img width="590" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/a4d222af-0123-407b-b37e-a26a392b9021">

###### Note: Please click on the image to enlarge it:


</br>
</br>
</br>
</br>

## LiquidInfrastructureERC20.sol

The ERC20 token acts as a share in the earnings from these assets, with distributions managed through the contract based on earnings from the associated NFTs.


Interactions with NFTs: The contract's ability to manage NFTs, withdraw earnings from them, and adjust the portfolio of managed assets indicates a close integration between tokenized infrastructure assets and their financialization through the ERC20 tokens. This is the part of the contract that should be best audited.

![image](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/9f701f27-fbad-445a-9f31-6e1756c2b366)

</br>
</br>
</br>
</br>

## LiquidInfrastructureNFT.sol
This contract integrates ERC721 standards to represent liquid infrastructure accounts with unique functionalities for managing ERC20 token thresholds and withdrawals.

 Initializes the contract by creating a unique ERC721 token with a name and symbol derived from the provided `accountName`. It concatenates "althea://liquid-infrastructure-account/" and "LIA:" with the `accountName` for the token's URI and symbol, respectively. The constructor mints a new token to the `msg.sender` with a predefined `AccountId` 


<img width="1123" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/dc66d70b-0a4d-4667-8d80-1b499883e750">

###### Note: Please click on the image to enlarge it:

</br>
</br>
</br>
</br>

## OwnableApprovableERC721.sol

The `OwnableApprovableERC721` contract provides modifiers for restricting function access either to the owner of a specific token ID or to an account approved by the owner. This is useful in scenarios where specific token-based permissions are needed on top of the standard ERC721 functionality. Analyzing the given contract, it already looks quite optimized for its purpose, but there are minor adjustments and improvements that can be made for clarity, gas optimization, and to adhere to best practices in Solidity development.


1. **Direct `ERC721` Calls**: The contract uses `ERC721(this).ownerOf(tokenId)` to check ownership, which is effectively calling the `ownerOf` function on itself. Directly using `ownerOf(tokenId)` without casting `this` would be cleaner and more gas-efficient.

2. **Modifier Reuse**: The `onlyOwnerOrApproved` modifier indirectly reuses logic from the `onlyOwner` modifier by checking for ownership. It then extends this check to include approved operators. While this modular approach is logical, it could potentially be streamlined by directly incorporating the ownership check into `onlyOwnerOrApproved`, thus avoiding the need for the separate `onlyOwner` modifier if all instances where `onlyOwner` is used could also logically allow for approved operators.


<img width="911" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/81dee262-8098-4076-9ebc-28ecac161312">

###### Note: Please click on the image to enlarge it:

</br>
</br>
</br>
</br>

## c) Test analysis
### What did the project do differently? ;
-   1) It can be said that the developers of the project did a quality job, there is a test structure consisting of tests with quality content.

### Test Content:
1. **Testing Setup and Tools**: The test suite utilizes Chai for assertions, Hardhat for the Ethereum development environment, and ethers.js for interacting with the Ethereum blockchain. Waffle is also used to extend Chai with Ethereum-specific matchers, providing a more expressive syntax for testing smart contract functions.

2. **Testing Scenarios**:
     - **Deployment**: Contracts are deployed using custom utility functions, ensuring that the test environment is set up consistently.
     - **NFT Management**: Tests confirm that only the owner can manage NFTs and asserts proper event emissions upon successful management actions.
     - **ERC20 Holder Management**: Tests validate that only approved holders can receive tokens, enforcing the contract's access control rules.
     - **Distribution Tests**: These tests simulate the distribution of ERC20 tokens to holders, including complex scenarios with random distributions and multiple NFTs.
     - **Ownership and Approval Tests**: Scenarios include transferring ownership of NFTs and revoking approvals when ownership changes.

### Test Quality:
1. **Assertion Depth**: The tests use a variety of assertions to check for state changes, event emissions, and access control enforcement. This indicates a comprehensive approach to validating contract functionality.
2. **Negative Testing**: The tests include scenarios where functions are expected to fail, such as trying to manage NFTs with a bad signer or minting to unapproved holders. This is a good practice as it ensures that the contract behaves as expected even in error conditions.
4. **Event Parsing**: Utility functions are used for parsing events, indicating a robust approach to confirm that the contracts emit the correct events with the expected values.

### Test Scope:
1. **Functionality Coverage**: The tests cover critical functionalities such as NFT management, ERC20 token distribution, and access controls. This suggests that the test suite is designed to verify the core business logic of the contracts.
2. **Access Control**: The test suite has a strong focus on access control mechanisms, such as ownership and approval. This is crucial for contracts that have restricted actions based on user roles.
3. **Edge Cases and Security**: There are tests that simulate random distributions and check for correct balances afterward, which is important for verifying the contract's reliability and security in varied scenarios.
5. **Inter-contract Interaction**: The tests cover interactions between the `LiquidInfrastructureNFT` and `LiquidInfrastructureERC20` contracts, ensuring that the integrated system works as intended.

### Comments and Recommendations:
- **Mocks and Stubs**: The test suite should consider using mocks for external contracts or modules that are not within the scope of the testable environment (e.g., Cosmos modules).
- **Testing Environment**: It is noted that the testing environment is limited to Hardhat, which means interactions that require a live network or cross-chain functionalities may not be adequately tested.
- **Contract State**: The tests should ensure that the state of the contract before and after test execution is consistent and isolated to avoid side effects.
- **Code Reusability**: Utility functions for deploying contracts, signing hashes, and parsing events are used, which is good practice for code reusability and readability.
- **Documentation**: Inline comments explain key aspects of the tests, but more detailed documentation could be beneficial for understanding the intended behaviors and potential test coverage gaps.


In general, the project team has done a good job in testing, the test suites seem well-structured with a strong focus on key functionalities and security aspects of the smart contracts.



### What could they have done better?





-  1) If we look at the test scope and content of the project with a systematic checklist, we can see which parts are good and which areas have room for improvement As a result of my analysis, those marked in green are the ones that the project has fully achieved. The remaining areas are the development areas of the project in terms of testing ;


<img width="785" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/2f1af16d-d629-4f1d-950a-b39a77c2d838">

Ref: https://xin-xia.github.io/publication/icse194.pdf

</br>
</br>

-   2) Overall line coverage percentage provided by your tests : 95 (As a generally accepted safety rule, 100% test coverage is recommended)


</br>
</br>



-  3) Test Tools/Technologies analysis of the project;

<img width="986" alt="image" src="https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/assets/104318932/724c94fa-db1e-41f7-adb8-c1670f83ab48">




</br>
</br>
</br>
</br>

## d) Security Approach of the Project

### Successful current security understanding of the project;

1- They manage the 1st audit process with an i Code4rena is a good choice for security that many eyes will look at


### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. 
https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

7- Emergency Action Plan
In a high-level security approach, there should be a crisis handbook like the one below and the strategic members of the project should be trained on this subject and drills should be carried out. Naturally, this information and road plan will not be available to the public.
https://docs.google.com/document/u/0/d/1DaAiuGFkMEMMiIuvqhePL5aDFGHJ9Ya6D04rdaldqC0/mobilebasic#h.27dmpkyp2k1z

8- ChainAnalysis oracle
With the ChainAnalysis oracle, OFAC interaction can be blocked so that legal issues do not arise


9 - As the project team, you can consider applying the multi-stage audit model.

<img width="498" alt="image" src="https://github.com/code-423n4/2023-12-ethereumcreditguild/assets/104318932/7ad49e46-4998-4bf2-830e-711039ea97a8">

Read more about the MPA model;
https://mpa.solodit.xyz/

10 - I recommend having a masterplan applied to project team members (This information is not included in the documents).
All authorizations, including NPM passwords and authorizations, should be reserved only for current employees. I also recommend that a definitive security constitution project be found for employees to protect these passwords with rules such as 2FA. The LEDGER hack, which has made a big impact recently, is the best example in this regard;

https://twitter.com/Ledger/status/1735326240658100414?t=UAuzoir9uliXplerqP-Ing&s=19


11 -Another security approach I can recommend is 0xDjango's security approaches in a project. This approach divides security into layers (Test, Automatic Tool, Individual Audit, Corporate Audit, Economic Audit) and aims to have each part done separately by experts in the field. I can also recommend this approach to you;

https://x.com/0xDjangoOnChain/status/1748402771684909094?s=20

</br>
</br>

## e) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md

- **[M-01] and [M-02]** point out critical security and access control oversights, such as preferring `_safeMint()` over `_mint()` for additional checks and missing burning access control. These are vital for contract integrity and preventing unauthorized actions.
- **[M-03]** mentions the unsafe use of `transfer()/transferFrom()` with `IERC20`, which could lead to reentrancy attacks or loss of funds. It's crucial to ensure external calls are handled securely.



</br>
</br>
</br>
</br>

## Centralization Risk

#### Owner Role:

Centrality Risk High: The Owner role is highly centralized since it is singular and possesses significant administrative powers. 

The `onlyOwner` modifiers use across various functions in the `LiquidInfrastructureERC20` and `LiquidInfrastructureNFT` contracts, as well as the use of similar access control mechanisms in the `OwnableApprovableERC721` contract. These modifiers restrict the execution of critical functions to the contract's owner or to addresses explicitly approved by the owner, indicating a centralized control model.

### Centralization Risk Summary:

1. **Single Point of Failure**: The heavy reliance on `onlyOwner` for key operational functions such as approving holders, managing NFT contracts, and setting distribution parameters introduces a single point of failure. If the owner's account is compromised, the entire system's integrity and assets could be at risk.

2. **Limited Permission Flexibility**: The current access model does not allow for granular permission management beyond the binary choice of owner or not-owner. This could limit the system's adaptability to more complex governance models or operational requirements.

3. **Upgrade and Maintenance Risks**: Centralized control over upgrades and maintenance could lead to downtime or unilateral changes that might not be in the best interest of all stakeholders.

5. **Trust and Transparency Issues**: Users must trust the owner not to act maliciously or incompetently. This trust model may not be suitable for all types of decentralized applications where transparency and equal stakeholder input are valued.

### Strategies to Mitigate Centralization Risk:

1. **Decentralize Ownership**: Implement a multisig wallet as the contract owner. This approach requires multiple parties to agree on transaction execution, significantly reducing the risk of unilateral actions due to compromise or malice.

2. **Use a DAO for Governance**: Transition to a decentralized autonomous organization (DAO) structure for critical decision-making processes. This could involve token holders voting on key actions like upgrading contracts, changing parameters, or managing assets.

3. **Role-Based Access Control (RBAC)**: Introduce more nuanced roles and permissions beyond the binary owner model. RBAC allows for specific functions to be accessible by users with certain roles, improving flexibility and security.

4. **Timelocks for Critical Operations**: Implement timelocks for sensitive operations, especially those related to upgrades or parameter changes. Timelocks ensure that proposed actions have a mandatory waiting period, during which they can be reviewed and potentially vetoed by the community.


```solidity

17 results - 3 files

liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol:
  105       */
  106:     function approveHolder(address holder) public onlyOwner {
  107          require(!isApprovedHolder(holder), "holder already approved");

  115       */
  116:     function disapproveHolder(address holder) public onlyOwner {
  117          require(isApprovedHolder(holder), "holder not approved");

  305          uint256 amount
  306:     ) public onlyOwner {
  307          if (_isPastMinDistributionPeriod()) {

  320          uint256 amount
  321:     ) public onlyOwner nonReentrant {
  322          _mint(account, amount);

  393       */
  394:     function addManagedNFT(address nftContract) public onlyOwner {
  395          LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);

  415          address to
  416:     ) public onlyOwner nonReentrant {
  417          LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);

  442          address[] memory _distributableERC20s
  443:     ) public onlyOwner {
  444          distributableERC20s = _distributableERC20s;

liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol:
   49       * @notice This NFT holds only 1 token in it, which is the Account token. Its Id is `1`.
   50:      * This Id is used for access control via the onlyOwner/onlyOwnerOrApproved modifiers.
   51       * See OwnableApprovableERC721 for more info.

  110          uint256[] calldata newAmounts
  111:     ) public virtual onlyOwnerOrApproved(AccountId) {
  112          require(

  202       */
  203:     function recoverAccount() public virtual onlyOwner(AccountId) {
  204          emit TryRecover();

liquid-infrastructure/contracts/OwnableApprovableERC721.sol:
   7  /**
   8:  * An abstract contract which provides onlyOwner(id) and onlyOwnerOrApproved(id) modifiers derived from ERC721's
   9   * onwerOf, getApproved, and isApprovedForAll functions

  13   *      // Only the owner of RockId gets to use this function, regardless of approval status
  14:  *      function throwRock() public onlyOwner(RockId) { ... }
  15   *
  16   *      // Either the owner, a sender approved for all the tokens the owner of PaperId, or a sender approved for specifically PaperId gets to call this
  17:  *      function wrapWithPaper() public onlyOwnerOrApproved(PaperId) { ... }
  18   * 

  23       */
  24:     modifier onlyOwner(uint256 tokenId) {
  25          require(

  34       */
  35:     modifier onlyOwnerOrApproved(uint256 tokenId) {
  36          // Get approval directly from ERC721's internal method
```


</br>
</br>
</br>
</br>


##  f) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | A lot of OZ contracts |-  Version `4.3.1` is used by the project, it is recommended to use the newest version `5.0.1`| 


</br>
</br>





### Time spent:
16 hours