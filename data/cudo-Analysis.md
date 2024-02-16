## 1. Introduction

This is the basic introduction about what the Protocol is.
Liquid Infrastructure is a protocol designed to facilitate the tokenization and investment in tangible assets with on-chain revenue streams.
LiquidInfrastructureNFTs offer versatility, capable of representing various assets such as routers engaging in Althea's pay-per-forward billing protocol, vending machines, renewable energy infrastructure, or electric car chargers.

## 2. Architectural Improvement

I will try to provide acrvhitectural improvement By contracts in sequence.
Firstly there are `three` contracts in scope.
- Here is the table of scope contracts with necessary information (i.e sloc).

<table>
<thead>
<tr>
<th id="type" style="text-align:left;"> Type </th>
<th id="file" style="text-align:left;"> File   </th>
<th id="logic_contracts" style="text-align:left;"> Logic Contracts </th>
<th id="interfaces" style="text-align:left;"> Interfaces </th>
<th id="lines" style="text-align:left;"> Lines </th>
<th id="nlines" style="text-align:left;"> nLines </th>
<th id="nsloc" style="text-align:left;"> nSLOC </th>
<th id="comment_lines" style="text-align:left;"> Comment Lines </th>
<th id="complex._score" style="text-align:left;"> Complex. Score </th>
<th id="capabilities" style="text-align:left;"> Capabilities </th>
</tr>
</thead>

<tbody>
<tr>
<td style="text-align:left;"><p>📝 </p></td>
<td style="text-align:left;"><p>liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol </p></td>
<td style="text-align:left;"><p>1 </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p>206 </p></td>
<td style="text-align:left;"><p>192 </p></td>
<td style="text-align:left;"><p>74 </p></td>
<td style="text-align:left;"><p>107 </p></td>
<td style="text-align:left;"><p>68 </p></td>
<td style="text-align:left;"><p><strong><abbr title="Initiates ETH Value Transfer">📤</abbr></strong> </p></td>
</tr>



<tr>
<td style="text-align:left;"><p>🎨 </p></td>
<td style="text-align:left;"><p>liquid-infrastructure/contracts/OwnableApprovableERC721.sol </p></td>
<td style="text-align:left;"><p>1 </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p>43 </p></td>
<td style="text-align:left;"><p>43 </p></td>
<td style="text-align:left;"><p>19 </p></td>
<td style="text-align:left;"><p>22 </p></td>
<td style="text-align:left;"><p>13 </p></td>
<td style="text-align:left;"><hr></td>
</tr>

<tr>
<td style="text-align:left;"><p>📝 </p></td>
<td style="text-align:left;"><p>liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol </p></td>
<td style="text-align:left;"><p>1 </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p>477 </p></td>
<td style="text-align:left;"><p>458 </p></td>
<td style="text-align:left;"><p>251 </p></td>
<td style="text-align:left;"><p>158 </p></td>
<td style="text-align:left;"><p>201 </p></td>
<td style="text-align:left;"><p><strong><abbr title="Initiates ETH Value Transfer">📤</abbr></strong> </p></td>
</tr>


<tr>
<td style="text-align:left;"><p>📝🎨 </p></td>
<td style="text-align:left;"><p><strong>Totals</strong> </p></td>
<td style="text-align:left;"><p><strong>7</strong> </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p><strong>726</strong>  </p></td>
<td style="text-align:left;"><p><strong>804</strong> </p></td>
<td style="text-align:left;"><p><strong>424</strong> </p></td>
<td style="text-align:left;"><p><strong>311</strong> </p></td>
<td style="text-align:left;"><p><strong>344</strong> </p></td>
<td style="text-align:left;"><p><strong><abbr title="Initiates ETH Value Transfer">📤</abbr></strong> </p></td>
</tr>

</tbody>
</table>



<a href="https://imgur.com/WeckO4n"><img src="https://i.imgur.com/WeckO4n.png" title="source: imgur.com" /></a>


#### 2.1 LiquidInFrastructureERC20.sol

As I've been reading this contract, according to my understanding The LiquidInfrastructureERC20 contract designed to enable investors to earn rewards from managed LiquidInfrastructureNFTs. It serves as an aggregation layer for investment in real-world assets, allowing holders to receive revenue from network activities represented by tokens.

but the overall architecture could be improve by incorporating ERC20 standards for fungible token functionality and integrates with ERC721 contracts for managing NFTs representing network infrastructure in more robust and secure way.

<a href="https://imgur.com/odMl6Ii"><img src="https://i.imgur.com/odMl6Ii.png" title="source: imgur.com" /></a>

#### 2.2 LiquidInfrastructureNFT

According to me the simplest explanation of this contract is The LiquidInfrastructureNFT contract controls a Liquid Infrastructure Account, representing infrastructure assets involved in Althea pay-per-forward networks. It interacts with the x/microtx module for revenue collection and withdrawal.

According to my observation Leverages ERC721 standards for NFT functionality and incorporates access control mechanisms for owner and approval-based functions.

<a href="https://imgur.com/pPf9b22"><img src="https://i.imgur.com/pPf9b22.png" title="source: imgur.com" /></a>

#### 2.3 OwnableApprovableERC721

In my conclusion The OwnableApprovableERC721 contract is an abstract contract providing access control modifiers for managing ownership and approval status of ERC721 tokens. It enhances ERC721 functionality by introducing modifiers to restrict certain functions to token owners or approved addresses.

As i've been doing walkthrough in this code base i found Inherits from ERC721 and extends its functionality by adding access control modifiers for ownership and approval status. Provides a standardized approach for implementing ownership-based access control in ERC721 contracts.

<a href="https://imgur.com/SJix3wh"><img src="https://i.imgur.com/SJix3wh.png" title="source: imgur.com" /></a>

## 3 Main Features Of Protocol

I'll try to conclude main features of protocol contract by contract in a sequence 

### 3.1 LiquidInFrastructureERC20.sol

- Token holders earn rewards from managed LiquidInfrastructureNFTs.
- Distribution of revenue from network activities to token holders.
- Locking mechanism during distribution periods to prevent token transfers, mints, and burns.
- Ability to mint tokens, withdraw revenue, and manage approved holders.
  
### 3.2 LiquidInfrastructureNFT

- Control of Liquid Infrastructure Accounts via NFTs.
- Management of revenue thresholds for x/microtx module interaction.
- Recovery process for lost accounts.
- Withdrawal of ERC20 balances accumulated from network revenue.

### 3.3 OwnableApprovableERC721

- `onlyOwner` modifier restricts functions to the owner of a specific token.
- `onlyOwnerOrApproved` modifier restricts functions to the owner or addresses approved by the owner.
- Simplifies access control management in ERC721 contracts, particularly for contracts with manually `enumerated tokens`.
## System Overview In Diagrams
Here is the overview how each and every componenet is intracting with each other in protocol.

<a href="https://imgur.com/TmDXaSo"><img src="https://i.imgur.com/TmDXaSo.png" title="source: imgur.com" /></a>

## 4 Security Improvement 

- Strengthening access controls to restrict sensitive operations.
- Ensuring only authorized entities can perform critical functions.
- Implementing safe token transfer mechanisms with thorough error handling.
- Minimizing reliance on external calls and validating inputs/outputs.
- Conducting rigorous testing and security audits.
- Considering timelock mechanisms for critical operations.
- Implementing continuous monitoring for suspicious activities.
#### 4.1 Switch test suite from hardhat to foundry 
The protocol is using hardhat instead of foundry.

Hardhat is a good testing framework but it does not provide enough coverage as compared of foundry.

Because foundry provides fuzzing and invariant testing which helps alot in covering and exposing edge cases.

- Implement `fuzz testing`
- Implement `invariant testing`
## 5 Systematic Risk / Centralisation Risk
If ownable or approveable address get compromise whole protocol will break and become useless.
Try using multisig for admins addresses.
Further more 
Centralization risks in these contracts:

1. Ownership concentration.
2. Single points of failure.
3. Dependency on external entities.
4. Governance centralization.

Mitigation involves decentralizing control, reducing reliance on single entities, and implementing community governance mechanisms.

## 6 Main Features of Codebase 
Here I try to render Main features of codebase in diagrams which would help developers and user in understanding about the main features and functionalities of protocol. 



<a href="https://imgur.com/4ypyWWF"><img src="https://i.imgur.com/4ypyWWF.png" title="source: imgur.com" /></a>

## 7 Approach Evaluating CodeBase 
#### Day 1 
- Cloned repo and set it up, had trouble setting it up xD, but it was good experience setting it up and protocol contracts was in good sequence and order.
- Tried to read overview on C4 homepage about the protocol 
- Read the documentations of the protocol 

#### Day 2 

- Read the main functionalities of the code and tried to understand it 
- Compare the code with documentation 
- Try to look where it does not work as intended 

#### Day 3 

- Made notes
- Made diagrams with <3 on Miro to understand the codebase in a better way 
- Wrote Analysis Report 



## Time Spent 

- 10 Hours of focused work 

### Time spent:
10 hours