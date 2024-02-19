# Analysis Report

## Preface

This audit report should be approached with the following points in mind:

1. The report does not include repetitive documentation that the team is already aware of.
2. The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
3. If there exists repetitive documentation (mainly in [Mechanism Review](#mechanism-review)), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Approach taken in evaluating the codebase

### Time spent on this audit: 14 hours

0-2 hours:
 - Understood the codebase and architecture of the contracts
 - Added inline bookmarks for gas optimizations and low-severity issues

2-5 hours: 
 - Adding possible HM severity issues
 - Asking sponsors questions not included in documentation
 - Examining possible edge cases/extreme values for input parameters

5-10 hours:
 - Filtering out inline bookmarks
 - Writing reports for HM severity issues
 - Drawing mental models of processes

10-12 hours:
 - Writing Gas and QA reports
 - Searching for possible optimizations for some more time

12-14 hours:
 - Writing Analysis Report
 - Reviewing findings

## Architecture recommendations

### Protocol Structure

The protocol is structured in a simplistic manner. There are 3 contracts: LiquidInfrastructureERC20, LiquidInfrastructureNFT and OwnableApprovableERC721. The LiquidInfrastructureERC20 can own/manage multiple LiquidInfrastructureNFT contracts. The LiquidInfrastructureNFT contracts could be standalone as well depending on their use case.

**A new model:**

The protocol could be made more simplistic if the ERC1155 standard is used. In this case, the contract that inherits ERC1155 would hold the LiquidInfrastructureNFTs. The LiquidInfrastructureNFT would be represented using the tokenIds, which itself could be further fractionalizable due to tokenIds being semi-fungible. All functions related to LiquidInfrastructureNFT would be included in the contract that inherits the ERC1155 contract.

**Where does the revenue come in from?**
 - The revenue in the original model first needs to be deposited to the LiquidInfrastructureNFT contract by the micro-tx module and then transferred to the LiquidInfrastructureERC20. 
 - This extra jump won't be required in case of the new model. The micro-tx module can look at the thresholds set by the owner of the tokenId using setThresholds() existing in the contract that inherits the ERC1155 contract.

**How does revenue distribution occur?**
 - Since the revenue ERC20 tokens are deposited by the micro-tx module into the ERC1155 contract itself, the distribution would also occur from the distribute() function existing in the contract.

**What will the ERC20 token holders hold?**
 - The original model required users to hold only LiquidInfrastructureERC20 tokens to earn revenue.
 - The new model can allow users to hold onto any tokens to earn revenue. The protocol can decide which tokens to consider. The team can also just use one standalone ERC20 contract token (as done in LiquidInfrastructureERC20) to achieve the same purpose as the original model.

This model would provide the same features/benefits as the original model, with an add-on of the tokenId/LiquidInfrastructureNFT being fractionalizable.

### What's unique?

1. Representation of RWA - Real world assets are tokenized using NFTs, which can provide revenue in multiple tokens to infrastructure holders. This incentivization opens up a new realm of earning opportunities to individuals who provide services on Althea and support the network by holding infrastructure tokens.
2. DEXes for infrastructure token holders - The team/protocol managers are expected to build a DEX, which would allow approved holders to swap and LP their tokens to one another. This creates a like minded community of holders supporting Althea by creating a market for them.
3. Regular holders - Although the infrastructure model seems to be inclining towards RWAs, there is still a place for regular holders/investors to profit of the revenue just by holding the infrastructure tokens in a decentralized manner.

### What ideas can be incorporated?

1. Creating a DAO for infrastructure token holders on Althea. This DAO would vote on proposals such as possible revenue token additions they might want to earn in and removal of any existing revenue tokens or maybe even adding/releasing NFTs that are not generating any revenue.
2. Providing revenue back to the NFT that provides highest revenue. The NFT when released would be sent to the original owner who created the NFT as a token of appreciation.
3. Revenue could be redirected towards a treasury as a small fee for the infra service the althea team provides. This could be used by the team to support projects on althea and provide them with grants.

## Centralization risks

The biggest centralization vector in the codebase is the owner. The owner has the following privileges currently:
1. Mint tokens to any address
2. Approve/Disapprove any holders
3. Add/remove managed NFT contracts
4. Update the revenue ERC20 tokens

Consider using a multi-sig for the owner role. If possible, consider rotating the owner role to new multi-sigs every 6 months for additional security.

## Resources used to gain deeper context on the codebase

1. [Althea documentation](https://github.com/althea-net/liquid-infrastructure-contracts/tree/main)
2. [Cosmos documentation](https://docs.cosmos.network/v0.50/build/modules/bank)

## Mechanism Review

### High-level System Overview

![Imgur](https://i.imgur.com/12XmuIg.png)

### Chains supported
 - Althea L1
 - Gnosis (might be deployed as per sponsor)
 - EVM chains (by other teams/maintainers)

### Mental models

#### Inheritance structure

Contracts marked other than blue are out of scope.

![Imgur](https://i.imgur.com/nXHZhva.png)

#### Recovery mechanism

![Imgur](https://i.imgur.com/hlpxJ2f.png)

#### Features of Protocol/Contracts

![Imgur](https://i.imgur.com/9R7szqk.png)

#### Features of Micro-tx module

![Imgur](https://i.imgur.com/qunh0GG.png)

## Systemic Risks/Architecture-level weak spots and how they can be mitigated

1. Contracts would not work on all chains due to tokens like USDT, USDC, DAI, WETH etc having different signatures and decimals on different chains. Before an Althea maintainer deploys contract on the other chain, check if the tokens in scope use the correct function signatures. For example, USDT does not return bool in its transfer() function. Another example, USDT, USDC uses 6 decimals while DAI uses 18 decimals.
2. Address poisoning - Currently the protocol allows 0 value transfers. This opens up doors to address poisoning attacks, which could cause users to send tokens to incorrect addresses.
3. Zero value burn calls allow inflating the holders array to cause permanent OOG errors. This attack can be launched by a non-approved holder as well and requires urgent action. To avoid this, do not push the 0 address to the holders arrays as well on burns.
4. Consider using mappings over arrays - Currently there are numerous instances that could cause the protocol to undergo frequent OOG errors. The managedNFTs, erc20Entitlements, distributableERC20s, holders, thresholdERC20s arrays are at the risk of causing OOG errors when used in for loops. 
5. Consider locking the setter function for distributable erc20 tokens during distribution period. This would ensure the owner cannot mess around with any revenue tokens for some of the users mid distributions.

## Issues surfaced from Attack Ideas in [README](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure#attack-ideas-where-to-look-for-bugs) 

1. Attack Idea - Errors in the distribution to holders, such as the ability to take rewards entitled to another holder
 - Although other holders cannot steal rewards, there are two cases related to reward distributions.
 - In case one of the holders is blacklisted for just one of the revenue tokens, they are devoid of receiving the remaining revenue tokens for which they are not blacklisted. This also prevents the remaining holders from receiving their revenue since the distribute() function reverts.
 - The balance/supply calculation is flawed since if balance < supply or balance uses decimals less than 18 (such as USDT, USDC), the entitlement would round down to 0, causing holders to receive no payments for that distribution period. Those tokens remain in the contract and are subject to the same issue in future distribution periods unless the balance >= supply.

2. Attack Idea - DoS attacks against the LiquidInfrastructureERC20 are of significant concern, particularly if there is a way to permanently trigger out of gas errors
 - There are 2 scenarios that lead to mints, burns and transfers being permanently DOSed.
 - In case 1, an attacker (approved or non-approved holder) can make 0 value burn calls to inflate the holders array. This would cause OOG in the for loop of _afterTokenTransfer() function.
 - In case 2, if the holders array grows too large, it would cause the OOG in the for loop of _afterTokenTransfer() function.

3. Attack Idea - Acquiring the ERC20 (and therefore rewards) without approval is another significant concern
 - There are no issues regarding this.

### Some questions I asked myself and other FAQ:

1. What are Liquid accounts?
 - Once Althea-L1 is deployed, real-world devices will have accounts on the chain. They will use these accounts to send and receive ERC20 tokens for a service, most likely because they are part of the Althea internet service protocol. Each device account can opt-in to becoming a Liquid account - they will have a LiquidInfrastructureNFT deployed and will automatically forward balances beyond a configured threshold to the NFT. Althea-L1 the chain will manage the flow of stablecoins from the device accounts to the LiquidInfrastructureNFT.

2. Is it possible for any functionality to DOS?
 - Yes, mints/burns/transfers are possible to DOS using 0 value burn calls

3. Are the contracts ERC20 and ERC721 compliant?
 - Yes, in most cases. There are some cases such as blocking of transfers during distribution that break the spec somewhat.

### Time spent:
14 hours