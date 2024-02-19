# Analysis Report

### Table of Contents

| Section                  |
|--------------------------|
| [Table of Contents](#table-of-contents) |
| [Audit Methodology](#audit-methodology) |
| [Systemic risks](#systemic-risks) |
|   - [Token distribution](#token-distribution) |
|   - [ERC20 tokens](#erc20-tokens) |
| [Centralization risks](#centralization-risks) |
| [Other considerations](#other-considerations) |
|   - [Code readability](#code-readability) |
|   - [Upgradable contracts](#upgradable-contracts) |
|   - [Test coverage](#test-coverage) |
| [Conclusion](#conclusion) |
| [Time spent](#time-spent) |



### Audit Methodology

Day 1
- Read through the audit details, including bot report and automated findings, gathered rough idea of what the project was about and the context
- Briefly glanced at the code, checked out and validated the transfer hooks that facilitate KYC as mentioned in the audit details

Day 2 - 3
- Looked closer at the transfer hooks `_beforeTokenTransfer()` and `_afterTokenTransfer()`, noticing some flaws. Wrote PoC tests for them to validate the findings, then wrote reports for them.
- Looked at how token distribution works, especially `distribute()`. Noticed a 'push' system in place to distribute the rewards, which could be a major flaw.
- Found many vulnerabilities in `distribute()`, wrote PoC tests and reports for all of them.
- Noticed other vulnerabilities including major loss of precision in `_beginDistribution()`, validated with tests and wrote report.
- Started planning improvement possibilities for the project, which are detailed in this analysis report.

Day 4 (Final Day)
- Finalized my findings and analysis report.
- Read through my findings again to make improvements, spot mistakes / improve clarity.


### Systemic risks

#### Token distribution
Currently, LiquidInfrastructureERC20 uses a 'push' approach to distribute tokens. This leads to many issues when distributing tokens:

- High gas cost, lower incentive: there are nested `for` loops in `distribute()`, looping through a specified number of holders distributing every distrubutable ERC20 token to them. Due to the `holders` array determining the order holders receive their reward, this can lead to lower incentives to retreive the reward. For example, if holder `B` is at the end of the `holders` array, and the array contains 100 holders, they must go through 99 other holders before receiving their token, which they have little incentive to do so. This can lead to holders at the end to wait for holders at the front to withdraw their rewards, or else they must pay gas costs for other holders.

- Lots of bugs in `distribute()`: the current implementation can be DoSed in many ways, and if a DoS happens, no one can retreive their reward, and token transfers will be locked until distribution finishes. If the function is permanently DoSed, this leads to loss of all rewards and even all infraERC20 tokens as they are locked in the contract. This inherently brings huge risks, especially since other ERC20 tokens are expected to be used which may have adverse effects, for example, USDT not returning a `bool`.

- Problems with `holders` array: this array keeps track of anyone with a positive infraERC20 balance. However bugs in `_beforeTokenTransfer()` and `_afterTokenTransfer()` lead to accounts with zero balance from existing, even the zero address from existing, as well as duplicate addresses not being removed. It's trivial for anyone to flood `holders` with tons of addresses, leading to unnecessarily high gas costs when distributing tokens.

- Scaling: the current implementation doesn't scale well as more holders are added, as the longer array leads to high gas costs to be payed. For future considerations, scaling and the amount of people expected to use the protocol should be taken into account, as well as the gas costs that they are expected to pay.

Instead of the current approach, I suggest a [pull over push](https://fravoll.github.io/solidity-patterns/pull_over_push.html) be implemented for users to withdraw their own rewards themselves, instead of the contract pushing out everyone's rewards.

To implement this, perhaps use a double mapping such as
```solidity
mapping(address => mapping(address => uint256)) public holderTokenRewards;
```
to track the holder's reward for each token (for example, `holderTokenRewards[holder1][usdt]` returns the amount of `usdt` token rewards `holder1` has). Then a `withdrawReward()` function should be implemented for holders to withdraw their rewards. As for when distributing starts, another function can be implemented for holders to call, which updates their respective reward based on their holdings.

Of course, this alternative approach may lead to holders forgetting to, or not claiming their rewards during distribution. If this is the case, `holders` can still be used, but only for the purpose of being called once during distribution where each holder is looped through and their reward for each token is updated on the `holderTokenRewards` mapping instead of being sent out, incurring much less gas. It should also be made clear that duplicate addresses in `holders` can be detrimental - could lead to duplicate rewards etc. Thus a `rewardClaimed` mapping could be used, mapping holder addresses to a boolean.

#### ERC20 tokens

The sponsor mentions:
> Liquid Infrastructure is expected to use ERC20 stablecoins like USDC, Dai, or USDT as revenue tokens. The protocol is expected to vet ERC20s and opt-in to their use to avoid issues with non-standard tokens (e.g. rebasing tokens).

However when testing, I found that USDT doesn't even work at all - `distribute()` reverts when USDT is used, due to it not returning a `bool` which is expected by the `IERC20` interface.

Before choosing an ERC20 token to use, it should be tested throughly with the project to ensure no errors occur, as many ERC20 tokens behave strangely which can affect the logic of the contracts.

Also, there is currently no way to retreive ERC20 tokens if they do get stuck in LiquidInfrastructureERC20. Hence, a sweeper role, which could be the owner, can be added to retreive tokens locked in a contract.

### Centralization risks

The deployer of the LiquidInfrastructureERC20 is the owner, and has permissions to perform critical actions such as approving/disapproving addresses and setting reward tokens. The owner can also take any funds in the NFT contract, and mint themselves a high amount of tokens to get all the rewards for themselves. Even if the owner is not malicious, it's entirely possible for their keys to be compromised. In that case, the whole contract as well as its users would be jeopardized. To mitigate this, the contract could require multiple different accounts to execute a critical action (such as Gnosis Safe), or implement timelocks for critical actions to be executed.

### Other considerations

#### Code readability

At times I found it difficult and confusing to read some of the codebase, such as this:

```solidity
File: LiquidInfrastructureERC20.sol
142:         bool exists = (this.balanceOf(to) != 0);
143:         if (!exists) {
144:             holders.push(to);
145:         }
```

It is a bit unintuitive to use double `!=`, and the variable can be removed and replaced with a comment for better readability, such as:

```diff
-       bool exists = (this.balanceOf(to) != 0);
+       // add `to` to `holders` if it has 0 balance
-       if (!exists) {
+       if (this.balanceOf(to) == 0) {
            holders.push(to);
        }
```

#### Upgradable contracts

If a vulnerability is discovered post-deployment of the contracts, there would be high cost to redeploy the contract as accounting such as holders and mappings must be ported over, incurring very high amounts of gas.

Therefore, if an [upgradable contract](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable) was used, the logic of the contract can put behind a proxy and if the contract's underlying code had to be changed, this can be easily done by pointing the proxy at a different address.


#### Test coverage

Although the sponsor states:

> - What is the overall line coverage percentage provided by your tests?: 95.09

a lot of the vulnerabilities could have easily been found with better testing. The current tests seem to merely test functionality of the protocol, and not testing bounds where things are likely to fail, such as transferring 0 amount.

In particular, test ERC20 tokens were created for these tests, and not tokens that are expected to be used, such as USDC, Dai, or USDT. This lead to the finding that the contract doesn't work with USDT, which could have been easily found by testing on-chain tokens.

As for the precision lost vulnerability, this was due to precision lost in the testing as well:

```ts
File: liquidERC20.ts
410:     let entitlement = BigInt(division / supply) * totalRevenue;
```

Adding more thorough tests that resemble as much of the environment and its actual use case will result in the contract being less likely to fail or have vulnerabilities.

### Conclusion

The protocol generally works and functions well. However, there are some areas for improvement such as considering a different implementation of reward distribution, reducing centralization risks and making the contract upgradable. As the project develops, it is important to keep writing thorough tests in order to both verify the functionality and sensitive portions of the protocol. Edge cases in tests can be effective in catching bugs, and should be tested to ensure full coverage. After current vulnerabilities and flaws are addressed, it is important to keep security as a high priority and ensure safety of the project by seeking more audits and

### Time spent
25 hours




### Time spent:
25 hours