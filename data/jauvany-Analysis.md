**Overview**

Althea Liquid Infrastructure is a protocol to enable the tokenization and investment in real world assets which accrue revenue on-chain, and will be deployed on the Althea-L1 blockchain after launch. The protocol consists of tokenized real world assets represented on-chain by deployed LiquidInfrastructureNFT contracts, and the LiquidInfrastructureERC20 token that functions to aggregate and distribute revenue proportionally to holders. LiquidInfrastructureNFTs are flexible enough to represent devices like routers participating in Althea's pay-per-forward billing protocol, vending machines, renewable energy infrastructure, or electric car chargers. Liquid Infrastructure makes it possible to automatically manage these tokenized assets and arbitrarily group them, creating ERC20 tokens that represent real world assets of various classes. 

**Scope**

> The engagement involved analyzing three (3) different Solidity Smart Contracts:
 
- LiquidInfrastructureERC20.sol: This ERC20 withdraws revenue from its managed LiquidInfrastructureNFTs and distributes proportionally to holders.
 
- LiquidInfrastructureNFT.sol: This ERC721 accumulates ERC20 balances and enables permissioned balance withdrawals.
 
- OwnableApprovableERC721.sol: This abstract contract provides modifiers based on ownership and approval status of ERC721 tokens.


# Codebase quality analysis
 

**Unique:* 
- Codebase carries out specific governance mechanisms that are uniquely designed for Althea’s specific use case e.g Compliance of LiquidInfrastructureNFT.sol with ERC721 standard and contain a single token with ID 1.

**Existing Patterns:* 
- The Althea Protocol adheres to common contract management patterns, such as the use of onlyOwner and Ownable.

**Strengths:*
- All contracts in scope use locked solidity pragma version as  recomended.

**Weaknesses:*
- Lack of Official documentation
- Incomplete test coverage (95.09%). 100% test coverage is recomended 
- Named imports of parent contracts are not well utilised. 
 
# Mechanism review

- The mechanisms implemented in the Althea ecosystem, including the ERC721 standard and the various modules for ownership management, execution and extension, are innovative and well-designed. They provide a comprehensive range of features and capabilities for NFTs. 
However, there are some potential issues and risks associated with these mechanisms. These issues should be carefully considered and mitigated to ensure the security and reliability of the ecosystem.
 
# Centralization risks
 
- A single point of failure is not acceptable for this project. Centrality risk is high in the project as the roles of ownable and onlyOwner detailed below has very critical and important powers: Project and funds may be compromised by a malicious or stolen private key onlyOwner msg.sender

File: liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

```solidity

32: Ownable, 
```
```solidity

106: function approveHolder(address holder) public onlyOwner { 
```
```solidity
116: function disapproveHolder(address holder) public onlyOwner { 
```
```solidity
306: ) public onlyOwner { 
```
```solidity
321: ) public onlyOwner nonReentrant { 
```
```solidity
394: function addManagedNFT(address nftContract) public onlyOwner { 
```
```solidity
416: ) public onlyOwner nonReentrant { 
```
```solidity
443: ) public onlyOwner { 
```
```solidity
463: ) ERC20(_name, _symbol) Ownable() {
```

# Systemic risks
 
- External Contract Dependencies: Althea relies on external contracts from Openzepplin. If any of these contracts have vulnerabilities, it would affect the protocol.

- Test Coverage: The test coverage provided by Althea is : 95.09% however, 100% tests coverage is recommended.

- Like any smart contract-based system, Althea is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol. 


# Documents 

There is no official documentation providing a solid overview of how Althea Liquid Infrastructure is structured and how its various aspects function. 

I would also recommend creating quality Medium articles, it’s a great way to provide an indepth look at many of the topics in the project and it is used by many blockchain projects.

# Automated Findings 

See automated findings [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md)

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/4naly3er-report.md).

# Architecture recommendations
 
> Althea Liquid Infrastructure’s architecture seems solid in general, none the less here are some areas that could be improved:

- Testing and Simulations: I recommend creating a live testnet app. Here is an
[example](https://app.opendollar.com/) from The Open Dollar protocol. Conduct thorough testing of all contracts and functions and simulations to understand how they will behave under various market conditions.

- Monitoring Recommendations: While audits help in identifying code-level issues in the current implementation and potentially the code deployed in production, the Althea Liquid Infrastructure team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete security assessment, the monitoring recommendations section raises several actions addressing trust assumptions and out-of-scope components that can benefit from on-chain monitoring.

- I recommend rewriting some of the tests in the codebase for this audit to use the actual contracts instead of mock addresses like in some cases. This will offer greater confidence during system deployment.

- I recommend creating an official documentation preferably through the official website’s subdomain with detailed parts of the protocol as well as asociated diagrams

- Financial Activity: Consider monitoring the token transfers over the bridge to identify: Transfers during normal operations to establish a baseline of healthy properties. Any large deviation, such as an unexpectedly large withdrawal, may indicate unusual behavior of the contracts or an ongoing attack. Transactions that revert. These may indicate a user interface bug, an ongoing attack or other unexpected edge cases.

**Gas Optimizations**

- Review data types: Analyze the data types used in your smart contracts and
consider if they can be further optimized. For example, changing uint256 to
uint128 or uint94 can save gas and storage slots.
- Struct packing: Look for opportunities to pack structs into fewer storage slots. By carefully selecting appropriate data types for struct members, you can reduce the overall storage usage.
- Use constant values: If certain values in your contracts are constant and do
not change, declare them as constants rather than storing them as state variables. This can significantly save gas costs.
- Avoid unnecessary storage: Examine your code and eliminate any unnecessary storage of variables or addresses that are not required for contract functionality.
- Storage vs. memory usage: When working with arrays or structs, consider whether using storage instead of memory can save gas. Using storage allows direct access to the state variables and avoids unnecessary copying of data.
- Replacing the use of memory with calldata for read-only arguments in external  functions.

**Other recommendations**

- Regular code reviews and adherence to best practices.
- Conduct external audits by security experts.
- Consider open sourcing the contract for community review.
- Maintain comprehensive security documentation.
- Establish a responsible disclosure policy for vulnerabilities.
- Implement continuous monitoring for unusual activity.
- Educate users about risks and best practices.
 
# Approach taken in evaluating the codebase
 
> My analysis of Althea Liquid Infrastructure Included understanding the architecture, mechanism, overall codebase and possible risks associated to the protocol. 

- Day 1: I spent time reading the website and audit main page in order to have an understanding of the protocol. 

- Day 2: I analyzed the codebase for better understanding, Performed a Mechanism review and investigated possible systemic risks, and centralization risks. 

- Day 3: I dedicated this day to coming up with possible Architecture recommendations after identifying possible risks and prepared the final analysis report.


**Conclusion**

The Althea ecosystem is a well-designed and innovative NFT platform offering a wide range of features and capabilities. However, there are areas where improvements could be made, particularly in terms of permission management, gas efficiency, and potential exploitation of certain modules. By addressing these issues,  Althea liquid infrastructure could become even more secure, efficient, and reliable.

### Time spent:
24 hours