# Analysis Report

| Sl.no | Particulars        |
|-------|--------------------|
| 1     | Preface            |
| 2     | Audit approach overview |
| 3     | Contest Overview           |
| 4     | Approach Taken in Evaluating the Codebase |
| 5     | Final Risk Verdict |
| 6    | Centralization Risks |
| 7    | How they can be mitigated |
| 8    | Gas Optimizations  |

## Preface
This audit report should be approached with the following points in mind:

* The report covers the basic ideology of the protocol that Althea aims to achieve through Athena and discusses the high-level usage and necessity of the Althea Liquid Infrastructure codebase.
* This report aims to provide end users of Liquid Infrastructure with information regarding the financial opportunities available for participation.
* It outlines the risks and architectural rebases that I felt necessary after auditing.

## Audit approach overview
Time spent on this audit: 3 days (32 hrs)

Day 1:
* Studying Althena whitepaper and Liquid Infrastructure.
* Studying contest overview.
* Reviewed LiquidInfrastructureERC20.sol, LiquidInfrastructureNFT.sol, OwnableApprovableERC721.sol
* Understanding roles and interactions to the contracts

Day 2:
* Made a layout of various scenarios to operate with
* Left questions in the codebase
* Reviewing deployment and test scripts
* Answering questions and filtering valid issues
* Reported

Day 3:
* Writing analysis report

## Contest Overview

There are two main contracts (LiquidInfrastructureNFT.sol, LiquidInfrastructureERC20.sol) and one access control contract (OwnableApprovableERC721.sol)

![image](https://ibb.co/wQPwFjY)
https://ibb.co/wQPwFjY

## Approach Taken in Evaluating the Codebase

### Initial Code Review and Questioning

1. **OwnableApprovableERC721.sol:** This contract consists of modifiers that help verify whether an NFT token is owned or approved to the caller. The implementation looks fine.

2. **LiquidInfrastructureNFT.sol:** Every time an Infrastructure is created, it mints an NFT with `accountId = 1`. Holding the Liquid Infrastructure NFT represents ownership of the Infrastructure, allowing the owner to:
    * Withdraw ERC20 balances from the contract
        * Withdrawals are possible to the token owner itself or to any arbitrary addresses.
    * Set thresholds for each ERC20 token.
    * Recover accounts.

**Questions I Asked Myself:**   

❓Where does the NFT contract get tokens from to perform withdrawals?   
❓What incentives do Infrastructure nodes get in return for holding and maintaining?   
❓What are the ERC20 thresholds, and what are they used for?   

3. **LiquidInfrastructureERC20.sol:**
    * Liquid Infrastructure owners create an ERC20 token representing the Infrastructure NFT.
    * The owner sets the list of addresses that can hold the Liquid Infrastructure Tokens.
    * Allowed holder addresses can mint tokens (the more minted, the more value).
    * Rewards accumulated over time are distributed to holders in the form of ERC20s to all holders.
    * A waiting period `MinDistributionPeriod` is enforced for executing consecutive distributions.
    * Owner-controlled array `_distributableErc20s` represents tokens accepted as rewards.
    * Add/remove managed NFTs.
    * Owner can withdraw the rewards from all managed NFTs:
        * This is used to transfer the ERC20 tokens accumulated as rewards in Liquid Infrastructure NFT to the Liquid Infrastructure ERC20 contract.
        * Transfer not more than the threshold amount of the ERC20 (managed by the owner).
    * Burning tokens is allowed for everyone who holds the Liquid Infrastructure tokens.
    * During distribution, the following functions are halted:
        * Burning
        * Minting
        * Withdraw from managed NFT
        * Distribution

**Questions I Asked Myself:**   
      
❓Why would someone mint and be a holder?   
❓Where are the distribution tokens coming from to the contract?   
❓Trust Assumptions:   
- Owner can allow/disallow holders anytime (also during distribution).
- Owner can modify `distributableERC20s` (also during distribution).

### Reading Tests
There are three example test tokens created, and the `test/` directory contains scripts that answer many of the questions to validate the risk.

### Answering Questions

**LiquidInfrastructureNFT (LI NFT):**

❓ Where does the NFT contract get tokens from to perform withdrawals?      
❕ The x/microtx module is used to conduct microtransactions, with receiving accounts accruing Cosmos Coins. Any excess amounts left after paying upstream costs will be converted from Cosmos Coins to ERC20s and deposited into the LINFT contract.       

❓ What incentives do Infrastructure nodes get in return for holding and maintaining?       
❕ They receive rewards as tokens in `distributableERC20s`.       

❓ What are ERC20 thresholds, and what are they used for?       
❕ Rewards accumulated by the LI NFT contract can be withdrawn by the owner or left with a threshold amount for withdrawal. Tokens can be transferred to the LI ERC20 contract.       

**LiquidInfrastructureERC20 (LI ERC20):**
❓ Why would someone mint and be a holder?       
❕ By holding this ERC20, owners are entitled to revenue from the network represented by the token. Additionally, the more tokens held by owners, the more value and rewards they can receive.       

❓ Where do the distribution tokens come from to the contract?       
❕ Revenue is gathered from managed LiquidInfrastructureNFTs by the protocol and distributed to token holders on a semi-regular basis.       

❓ Trust Assumptions:       
1. The owner can allow/disallow holders at any time, including during distribution.       
2. The owner can modify `distributableERC20s`, also during distribution.       

### Trust Assumptions in-depth

1. When the owner can disallow holders:
    - No significant risk. Holders can't receive their rewards, but they can burn their tokens anytime.

2. Owner overrides `distributableERC20s`:
    - If the owner stops users from burning and all accumulated rewards are locked, as distribution never completes, there's a risk.
    - If the owner distributes rewards to a single person, it could lead to centralization concerns.
    - If the owner overrides distributable ERC20 tokens with a token whose contract has a balance of 0, the calculation of reward tokens to be distributed will result in a non-zero value, leading to a contract revert when distribution is called.

3. ERC20 listed as distributable blacklists a holder:
    - If the owner makes a particular blacklisted token not allowed, it can continue distribution, but it's unfair that a holder blacklisted for a single token is made not to receive all the tokens.

### Final Risk Verdict:
1. Reliance on trusting the owner seems to be a major flaw and is assumed to be a point of failure. Although the owner or token holder isn't at serious financial risk, this isn't desired for the correct functionality of the protocol.
2. The code is missing general ERC20, ERC721 safe practices like using `safeTransfer`.
3. Various ERC20 tokens must be assumed to be performing fine with the protocol. It doesn't handle special tokens such as blacklisting or fee-on-transfer as reward tokens.

## Centralization Risks:
1. Liquid Infrastructure NFT's owner is trusted to send the share of rewards to the associated ERC20 token contract through which token holders receive rewards.
2. Liquid Infrastructure's owner can halt the distribution process at will.
3. Token holder addresses can be managed by the owner anytime. This can be a defense mechanism in emergency cases and cause no loss to holders.

### Userflow:

1. Alice wants to invest in a real-world infrastructure project represented by a managed Liquid Infrastructure NFT (NILNFT) called "Internet Node" that provides internet services to a community.

2. Bob is the owner of the "Internet Node" NILNFT and has deployed the LiquidInfrastructureNFT contract to control it. The NILNFT has a unique ID associated with it, and Bob has set thresholds for ERC20 tokens that determine how much of each token should be left in the associated x/bank account.

3. Alice discovers the investment opportunity and decides to participate by acquiring tokens from the "Internet Node" project. She interacts with the LiquidInfrastructureERC20 contract deployed by Bob.

4. Bob has already added the "Internet Node" NILNFT contract address to the ManagedNFTs collection in the LiquidInfrastructureERC20 contract, indicating that revenue generated by the "Internet Node" will be distributed to token holders of the associated ERC20 token.

5. Alice purchases ERC20 tokens from the LiquidInfrastructureERC20 contract, becoming a token holder. By holding these tokens, she is entitled to a share of the revenue generated by the "Internet Node" NILNFT.

6. Bob periodically triggers a distribution of revenue generated by the "Internet Node" NILNFT. This distribution is processed by the LiquidInfrastructureERC20 contract, which calculates the entitlement of each token holder based on their token balance.

7. The LiquidInfrastructureERC20 contract distributes revenue in the form of ERC20 tokens to all token holders, including Alice, according to their entitlements.

8. Alice receives ERC20 tokens as revenue from the "Internet Node" NILNFT, reflecting her share of the revenue generated by the infrastructure project.

9. Bob may also withdraw balances from the "Internet Node" NILNFT using the withdrawFromManagedNFTs function in the LiquidInfrastructureERC20 contract, transferring excess ERC20 token balances from the NILNFT to the LiquidInfrastructureERC20 contract for distribution to token holders.

10. In case of a lost device or wallet, Bob can initiate a recovery process for the "Internet Node" NILNFT using the recoverAccount function in the LiquidInfrastructureNFT contract. This process transfers all balances from the x/bank account associated with the NILNFT to the LiquidInfrastructureNFT contract, allowing Bob to withdraw the balances later.

![Userflow](https://ibb.co/b6B36fX)

https://ibb.co/b6B36fX

## How they can be mitigated

Function `setDistributableERC20s()` can be paused during distribution to avoid single point of failures.
There might be a scenario where tokens need to be changed during distribution, but it seems invalid assumption as once distribution started the `erc20EntitlementPerUnit` are cached and never calculated again
```solidity
// File: LiquidInfrastructureERC20.sol
if (!LockedForDistribution) { //>  once distribution started tokens can be changed but further calculations work on cached entitlements
require(
_isPastMinDistributionPeriod(),
"MinDistributionPeriod not met"
);
_beginDistribution(); //> entitlements calculated here
}
// ...
for (uint j = 0; j < distributableERC20s.length; j++) {
IERC20 toDistribute = IERC20(distributableERC20s[j]);
uint256 entitlement = erc20EntitlementPerUnit[j] * this.balanceOf(recipient);
if (toDistribute.transfer(recipient, entitlement)) { //> if contract doesn't hold funds for token - reverts
receipts[j] = entitlement;
}
}
// ...
```

Fix:
```diff
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
+       require(!LockedForDistribution, "distribution in progress");
        distributableERC20s = _distributableERC20s;
    }
```

## Gas Optimizations:
Apart from bot race results here are few optimizations

Remove redundant code in `_beginDistribution()` to save gas
- If distribution is called for the first time `erc20EntitlementPerUnit` will be empty and no external/public functions initializations available. And when distribution completed `_endDistribution` ensures to make erc20EntitlementPerUnit empty. So the check can be removed



### Time spent:
32 hours