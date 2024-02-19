# Althea Liquid Infrastructure Analysis Report

## Preface 

This report is a summary of the analysis of Revolution Protocol's Codebase.
- This report is not an extension of the documents/Info provided by Althea Protocol.
- This report provides a high-level overview of the codebase and its security implications and some edge cases missed by developers and some suggestions to improve the codebase.
- This report aims to provide value to the sponsors and the developers of Althea Protocol.

## Brief Intoduction And Overview 

`Althea-L1` introduce a unique concept of Liquid Infrastructure, which is a protocol designed to facilitate the tokenization and investment in real-world assets that generate revenue on the blockchain. This protocol involves the creation of tokenized versions of tangible assets using LiquidInfrastructureNFT contracts, alongside the issuance of the LiquidInfrastructureERC20 token. This ERC20 token serves the purpose of aggregating and fairly distributing revenue to its holders. The versatility of LiquidInfrastructureNFTs allows them to represent a wide array of assets, including routers engaged in Althea's billing protocol, vending machines, renewable energy infrastructure, or electric vehicle charging stations. Through Liquid Infrastructure, managing these tokenized assets becomes automated, enabling the creation of ERC20 tokens that represent various classes of real-world assets.

Currently The Scope Contracts Are : 

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
<td style="text-align:left;"><p>contracts/LiquidInfrastructureNFT.sol </p></td>
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
<td style="text-align:left;"><p>contracts/OwnableApprovableERC721.sol </p></td>
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
<td style="text-align:left;"><p>contracts/LiquidInfrastructureERC20.sol </p></td>
<td style="text-align:left;"><p>1 </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p>477 </p></td>
<td style="text-align:left;"><p>458 </p></td>
<td style="text-align:left;"><p>251 </p></td>
<td style="text-align:left;"><p>159 </p></td>
<td style="text-align:left;"><p>201 </p></td>
<td style="text-align:left;"><p><strong><abbr title="Initiates ETH Value Transfer">📤</abbr></strong> </p></td>
</tr>

<tr>
<td style="text-align:left;"><p>📝🎨 </p></td>
<td style="text-align:left;"><p><strong>Totals</strong> </p></td>
<td style="text-align:left;"><p><strong>3</strong> </p></td>
<td style="text-align:left;"><hr></td>
<td style="text-align:left;"><p><strong>726</strong>  </p></td>
<td style="text-align:left;"><p><strong>693</strong> </p></td>
<td style="text-align:left;"><p><strong>344</strong> </p></td>
<td style="text-align:left;"><p><strong>288</strong> </p></td>
<td style="text-align:left;"><p><strong>282</strong> </p></td>
<td style="text-align:left;"><p><strong><abbr title="Initiates ETH Value Transfer">📤</abbr></strong> </p></td>
</tr>

</tbody>
</table>

## Codebase Quality 

After a deep walkthrough of Althea-Liquid-Infrastructure codebase, a well written and simplicity of code is observed. 
As in `LiquidInfrastructureNFT.sol` Which transfer all the revenue generated to `LiquidInfrastructureERC20` contract, the transferring complexity is handled very clearly and in a simple way, no headaches in understanding the code.

While in contrast there was some concepts in `LiquidInfrastructureERC20.sol` contract that took time to grasp because of complexity.As the code is inheriting transfer function from a libraries and internal functions called by that transfer function was in the main contract that led many auditors to the confusion as well. And this contract was also had some weakspots that are sure to be look up by the developers to improve the codebase Which will be discussed in Weakspots section.

## Approach Taken While Evaluating The Codebase

For auditing this codebase, I've adopted a mixture of `Blackboxing` & `SpeedRun` approach that is a combination well structured on focuing on just the main features of the protocol. i.e. Focusing on `ASSETs` Main `Actors` (Power Of Role & Normal Users), and `Action` (When and what can be done on what for whom) & Understand the protocol/expected scenarios through tests, and play around with them to break the developer assumptions and spot a bug as well which was submitted and It's affecting the Protocol Actors and Funds as well.

1. ##### Overview Page On C4: 

Started Auditing Althea Protocol By Opening the [Audit Page](https://code4rena.com/audits/2024-02-althea-liquid-infrastructure) on C4 and read the overview of the protocol and the scope of the audit.

Contest page On C4 Gave ideas and introduction about the protcols, Known issues and Attack vectors to look for in the codebase which helped me to focus on the main functionalities and features of the protocol.

2. ##### Code Review:

Cloned repo from Github and understood the basic structure of the codebase and codebase architecture was clean and well structured.

3. ##### Codebase Understadings: 

- LiquidInfrastructureERC20.sol

I started from the most complex part of the audit i.e. [LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol) and tried to understand all the components and the logic of this contract and how it's performing all the major functionalities and how it's interacting with other contracts and how it's handling the funds and the revenue generated by the `LiquidInfrastructureNFT.sol` contract.

- LiquidInfrastructureNFT.sol

Then I moved to the [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol) and tried to understand the logic of this contract and how it's handling the revenue and how it's interacting with the `LiquidInfrastructureERC20.sol` contract.

Understanding this contract was easy because all it was doing is transferring the revenue to the `LiquidInfrastructureERC20.sol` contract which was very solid and robust.

- OwnableApprovableERC721.sol

This contract was a solid contract having only modifiers or restrictions that are implement in other two contracts.

4. ##### 4naly3er and Known Issues 

Reading 4naly3er report and known issues found by bot is a great step to locate the weak spots in the codebase. After a walkthrough of codebase i was able to understand the known issues context and how they were impacting, That gave me more attack surface and ideas to look for in the codebase.

## Testing 

As I was applying `Blackboxing` approach in this audit, so playing with the tests was the main part of the audit. But, unfortunately, test suite was in `hardhat` which is a great testing framwork though but I prefer foundry for testing and also know foundry testing. 

1. ##### Integrating Foundry Testing

I integrated [foundry testing](https://book.getfoundry.sh/config/hardhat?highlight=hardhat#adding-foundry-to-a-hardhat-project) in this hardhat codebase by following these steps;

- [FoundryUp](https://book.getfoundry.sh/getting-started/installation) must be installed before integrating foundry testing in the hardhat project.
- inside Hardhat project working directory:
- with command `npm i --save-dev @nomicfoundation/hardhat-foundry` Install the hardhat-foundry plugin.
- Them, Added `require("@nomicfoundation/hardhat-foundry");` to the top of your hardhat.config.js file.
- Ran `npx hardhat init-foundry` in  terminal. It generate a foundry.toml file based on this Hardhat project’s existing configuration, and installed the forge-std library.

2. ##### Running Tests

The integration with foundry was already done but i wanted to execute the hardhat tests written by Althea developers. 

I executed those tests and tests were short and basic. 
Covered the main aspects of the protocol functionalities and features, but was unit tests so couldn't expose any edge cases or bugs in the codebase.

I tested two or three main components of the protcol, set up the basic foundry environment by inheriting all the contracts files and `forge-std` files.

Tried the `push pop` mechanism of the protocol and `deletion` of `erc20Entitlements` that if it generates incorrect or corrupted bytes.

3. ##### Static Analysis

- Slither 

I ran slither on the codebase and it found some minor issues that all were reported by Bot and 4naly3er as well.

- Aderynn

A new static analysis tool developed by Cyfrin, tested that out but it, too, was unable to find any major issues in the codebase.

4. ##### Testing Conclusion 

After testing the codebase both in hardhat and foundry multiple aspects came up.

- Test suite used by developers was very basic and was not able to expose any edge cases or bugs in the codebase.
- Test cases were only exposing developer assumptions which doesn't provide any value while handling edge cases.
- Employing Slither for static analysis showcased the team's proactive security stance, detecting and resolving vulnerabilities prior to deployment.

During the audit process, my main objective was to uncover disparities between the project's intended functionality and its actual implementation, evaluate adherence to security standards, and assess the overall strength of the codebase. This comprehensive audit was built upon meticulous code examination, familiarity with known vulnerabilities, analysis of external feedback, and thorough testing.

## Architectural Review of Althea Liquid Infrastructure

There are some `State Diagrams` and `Sequence Diagrams` in a contract by contract sequence that are missing in the documentation of the protocol. These diagrams might help to understand the protocol and its functionalities and features.

1. #### LiquidInfrastructureERC20.sol

State Diagram of How the `LiquidInfrastructureERC20.sol` contract is handling the revenue and how it's distributing the revenue to the holders and how it's interacting with the `LiquidInfrastructureNFT.sol` contract.

<a href="https://imgur.com/o6tnakW"><img src="https://i.imgur.com/o6tnakW.png" title="source: imgur.com" /></a>

Main functionalities and featues of the `LiquidInfrastructureERC20.sol` are :

The contract is **deployed** by the **owner**.
owner can **approves** a **holder** to **receive** the ERC20 token.
owner can **disapproves** a **holder** from **receiving** the ERC20 token.
owner can **initiates** a **distribution** to the **holders**. The contract **checks** the **entitlement** of each **holder** and **transfers** the corresponding amount of ERC20 tokens to them.
The owner can **mints** new **tokens** and **distributes** them to all **holders**.
owner can **adds** a new **NFT contract** to be **managed** by the ERC20 contract.
0wner can **releases** an NFT contract from being **managed** by the ERC20 contract.
owner can **sets** the ERC20 tokens that can be **distributed** from the **managed NFTs** to the **holders**.

- External and Internal Calls of `LiquidInfrastructureERC20.sol` contract.

<a href="https://imgur.com/eyixT1U"><img src="https://i.imgur.com/eyixT1U.png" title="source: imgur.com" /></a>

2. #### LiquidInfrastructureNFT.sol


<a href="https://imgur.com/dBXJ5qo"><img src="https://i.imgur.com/dBXJ5qo.png" title="source: imgur.com" /></a>

owner `initiates` the deployment of the contract, prompting the `minting` of the `AccountId` token.
owner establishes the thresholds for the contract.
owner `withdraws balances` from the contract. The contract verifies the balance of each ERC20 token and transfers it to the owner.
owner begins the account recovery process.

- External and Internal Calls of `LiquidInfrastructureNFT.sol` contract.

<a href="https://imgur.com/olu3No9"><img src="https://i.imgur.com/olu3No9.png" title="source: imgur.com" /></a>

#### 1.1 How protocol Contracts are Interacted with Each Other 

State Diagram of How the `LiquidInfrastructureERC20.sol` contract is interacting with the `LiquidInfrastructureNFT.sol` contract.

<a href="https://imgur.com/D2n1Kan"><img src="https://i.imgur.com/D2n1Kan.png" title="source: imgur.com" /></a>

## What Protocol Did Unique?

New concept has been introduced of tokenizing real world devices on chain. 

1. **Innovative Concept**: The protocol introduces a novel concept of enabling tokenization and investment in real-world assets that generate revenue on-chain. This bridges the gap between traditional assets and blockchain technology, offering new avenues for investment.

2. **Diverse Asset Representation**: LiquidInfrastructureNFTs are not limited to representing just one type of asset. They demonstrate creativity by being adaptable to represent a variety of real-world assets such as routers, vending machines, renewable energy infrastructure, or electric car chargers. This versatility expands the potential use cases for the protocol.

3. **Automated Asset Management**: Liquid Infrastructure offers automated management of tokenized assets, which streamlines processes and reduces manual intervention. This innovative approach allows for efficient management and tracking of assets on the blockchain, enhancing transparency and accessibility.

4. **Dynamic Asset Grouping**: The ability to arbitrarily group tokenized assets and create ERC20 tokens representing various classes of real-world assets showcases a dynamic approach to asset management. This flexibility enables investors to tailor their portfolios according to their preferences and investment strategies, adding a layer of customization to the investment process.

5. **Revenue Distribution Mechanism**: The LiquidInfrastructureERC20 token's function to aggregate and distribute revenue proportionally to holders demonstrates a creative way of incentivizing participation in the ecosystem. This mechanism promotes investor engagement and provides a tangible benefit for holding the token, thus encouraging long-term participation in the protocol.

## WeakSpots and Improvements Protocol Should Make

I found multiple weakspots most relatively Info/Refactoring and will try to write down contract by contract. 

- LiquidInfrastructureERC20.sol

1. **Use of for loop for element removal**: In releaseManagedNFT, a for loop is used to find and remove an NFT from ManagedNFTs, with a time complexity of O(n). Suggest using a mapping for faster O(1) access.

2. **Lack of input validation**: addManagedNFT doesn't validate if nftContract address is valid or already in ManagedNFTs, leading to potential issues.

3. **Gas limit potential**: distributeToAllHolders and withdrawFromAllManagedNFTs may hit gas limit with large arrays. Suggest chunk processing over multiple transactions.

4. **Missing events**: Functions like mint, burn, etc., don't emit events, hindering contract activity tracking.

5. **Inefficient storage**: Redundant data structures like HolderAllowlist mapping and holders array could be consolidated for efficiency.

6. **No holder view interface**: Lack of public function to view holders affects transparency and auditing.

7. **No distributable ERC20 view function**: Users can't view distributableERC20s, hindering transparency.

8. **No MinDistributionPeriod update**: Missing function to adjust MinDistributionPeriod for future frequency changes.

9. **No ManagedNFTs update**: Absence of function to update ManagedNFTs array for future adjustments.

10. **No HolderAllowlist update**: Missing function to update HolderAllowlist mapping for future holder changes.

- LiquidInfrastructureNFT.sol

1. **Array Should No Be Empty**: In the setThresholds function, a check ensures that the lengths of newErc20s and newAmounts are equal, but no check ensures these arrays are not empty. Consider adding a check to ensure they contain at least one element.

2. **Error messages**: Some require statements lack error messages, hindering debugging. Consider adding error messages to all require statements.

3. **Redundant code**: The withdrawBalances and withdrawBalancesTo functions are nearly identical. Refactoring to remove redundancy is advisable.

4. **Use of delete**: In setThresholds, delete clears thresholdErc20s and thresholdAmounts arrays unnecessarily as they're overwritten in subsequent loops, saving gas if removed.

5. **Function visibility**: _withdrawBalancesTo is marked internal but could be private since it's not used in derived contracts.

6. **Event emission**: SuccessfulWithdrawal is emitted after ERC20 transfers, risking non-emission if transfers fail. Consider emitting the event before transfers for consistent emission.

7. **Use of IERC20.transfer**: IERC20.transfer transfers ERC20 tokens; using IERC20.safeTransfer ensures successful transfer.

8. **Lack of function to update thresholdErc20s and thresholdAmounts**: Missing function to update thresholdErc20s and thresholdAmounts arrays, useful for future threshold adjustments.

9. **Lack of function to view thresholdErc20s and thresholdAmounts**: No public function to view thresholdErc20s and thresholdAmounts arrays, impacting transparency and auditing.

## Risk Related To The Protocol

Various risks are explained below

<a href="https://imgur.com/AHhhqSo"><img src="https://i.imgur.com/AHhhqSo.png" title="source: imgur.com" /></a>

- Lack of Access Modifier

Most of the protocol functions are public without having any access modifier or any restiction about who can call these function even one of the important key function of the prtocol (reported that as finging).

- Deprecated Libraries Can Affect The Protocol 

As addressed by Sponsor in Discord Chat and according to `packages.json` protocol is using 
`openzeppelin/contracts@4.3.1` which is very old and unsafe version of Oz to be used.
`OZ 4.9.0` must be used to avoid any vulnerabilities and to make the protocol robust against libraries attacks.

## Time Spent 

1. Joined Contest At 4th Day
- Day spent on Understanding the protocol and its functionalities and features.

2. Day 5 
- Made notes and point out some audit tags on weak spots in code
- connected weak spots to find out some attack surface and vulberabilities in the codebase.

3. Day 6 
- Write Reports For Finding
- Made some diagrams to understand the protocol and to use in the anlysis report as well
- Submit Findings and Analysis Report

Total Hours 

20 Hours



### Time spent:
20 hours