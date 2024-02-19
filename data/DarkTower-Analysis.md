# Analysis Report

## 1. Preface

This analysis report has been approached with the following key points and goals in mind:

 - This report is devoid of any redundant documentation the protocol is already familiar with.
 - It is engineered towards provision of valuable insights and edge cases the protocol left unnoticed.
 - Diagrams are simple to grasp for first-time users who stumble upon this report in the future.

## 2. Our review process of the codebase

**D1-2:**
 - Understanding of the protocol's concept
 - Mapping out security and attack areas
 - Keeping track of all areas with notes on contracts in-scope

**D2-4:**
 - Brainstorm possible reachability of undesired states
 - Test the areas identified
 - Further review of contract-level mitigations

**D4-6:**
 - Apply mitigations
 - Draft & submit reports

## 3. Architectural Recommendations

### What are the unique selling points?
  - Real World Asset Tokenization - There exists a subset of protocols already doing real-asset tokenization as NFTs in the blockchain space but Althea takes another approach at this with 1. Their L1 blockchain facilitating it, 2. Unique NFTs with constrained supply of such tokenized forms of the underlying asset.
 - Tokenized Assets as a group - These tokenized asset NFTs can be an underlying single asset for the corresponding Liquid ERC20 token or be many real world assets tied to just one Liquid ERC20. This is the 1 to many feature. (1 ERC20 with 2 or more underlying NFT assets)
 - Rewards for holding the asset backed ERC20 - Holders are whitelisted to get and hold these ERC20 tokens. Think about it like shares. Holding 50 shares of Apple assuming Apple rewarded all holders of such shares with USDC according to their share factor.

### Comparison
 - The first difference between Althea and NMKR is that the former tokenizes real world assets while the latter tokenizes just about anything and a marketplace ensues to sell the NFTs.

 - There's no ERC20 token representing holding of an NFT in the NMKR protocol. You hold the direct NFT instead. In Althea, you don't hold the NFT. You hold the ERC20 tokens. You are more like an investor instead of a price speculator.

 - Each holder of the ERC20 token in Althea gets rewarded in the reward tokens e.g USDC relative to their shares. No such incentive exists within the NMKR protocol which is instead a one and done protocol.

### What ideas can be implemented?
 - Decentralization of Holders: Currently, the one single biggest risk of this protocol is centralization. Becoming a holder of the ERC20 typically happens in a whitelist process. Allowing any address to become a holder of the ERC20 is a good way to increase adoption.

 - Whitelisted distributable ERC20 reward tokens: As is, the owner of the deployed ERC20 determines which token the rewards for holders will be distributed in. This is good but can be better because just as easy as it is to tokenize the reward NFT, deploy ERC20 for the underlying assets, it's also easy to setup fake tokens that cannot be distributed or even tokens with privileges that can render reward distribution to fail at any time. It's better the distributable tokens are set by the protocol and when an ERC20 owner tries to add a new distributable token, the token being added should be in the whitelisted array of tokens the protocol sets as accepted. This enforces a check and verify structure for the reward token.

## 4. Centralization Risks

There are two centralization points in the protocol:

1. The `LiquidInfrastructureERC20` owner who has control over the ERC20 contract. They have privileged actions such as setting the list of distributable reward tokens, removing and adding addresses who can hold the ERC20. This can be utilized as a multisig address to further decentralize the privileges of this single owner.

2. The `LiquidInfrastructureNFT` presents the similar risks but we think the more interesting contracts are always prone to weird behaviors which means the centralization risks of the NFT owner is far less compared to the ERC20 owner. For examole, the withrawal of funds to yourself (owner) can be considered design and won't compare to the ability to enlist fake reward tokens in the Liquid ERC20 contract.


## 5. Diagrams

![Entry point](https://rexjoseph.github.io/images/althea-entry.png)

### Some interesting call path diagrams for the users

With just 3 contracts in scope, the call traces of the codebase are relatively simple to understand.

![Call graph](https://rexjoseph.github.io/images/althea-call-graph.png)


## 7. Some weak spots within the codebase and mitigations
 - Removal of zero-value holders - Currently if after you make a transfer to another holder, your balance drops to zero, you're no longer a holder but it doesn't check if the receiving holder's address has a non-zero value. It sets them up as a holder if they weren't previously regardless. Anyone can use this oversight to inflate the holders array size.

 - Push technique for rewards - Currently rewards are distributed to holders of the ERC20. You can set an index to start distribution at but another way of implementing a better reward distribution process is by having the reciever take it themselves and keep track of each accumulated reward for all holders.

 - Frontruns - The protocol seems to be vulnerable to frontrun attacks. Be it for grabbing another name of an NFT or ERC20 token causing a scenario of confusion for which address is real or not with more than one project using the same name and symbols.

 - Phishing attacks - Anyone can deploy `LiquidInfrastructureERC20` & `LiquidInfrastructureNFT` tokens. Attackers can create similar tokens with names and symbols resulting in different owners. This can fool users via social engineering tactics in public communities.

### Time spent:
24 hours

### Time spent:
24 hours