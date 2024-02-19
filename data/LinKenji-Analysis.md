## Tokenize real-world revenue generating assets to enable collective investment

Specifically, the LiquidInfrastructureERC20 and LiquidInfrastructureNFT contracts aim to:

- Represent real-world infrastructure and utilities that generate revenue on-chain
- Allow tokenized ownership of these assets through NFTs
- Facilitate collective investment in diverse assets through an aggregated ERC20 token
- Automate distribution of accrued revenues to token holders  

This enables “liquidization” of traditionally illiquid assets - like electric vehicle chargers, vending machines, solar installations etc. By tokenizing these assets on-chain, it allows for fractionalized ownership, investment exposure, collective management, and built-in revenue distribution.

Some examples of real-world assets mentioned:

- Internet routers providing bandwidth in exchange for payments
- Electric vehicle charging stations receiving usage fees
- Vending machines monetized through on-chain micropayments
- Solar/wind installations generating saleable renewable energy

The protocol aims to "liquefy" real-world revenue streams to unlock shared ownership models.

### Architecture

The Liquid Infrastructure protocol consists of two core smart contracts:

- [LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol): An ERC20 token that aggregates revenue and distributes it to holders
- [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol): An NFT representing a real-world revenue generating asset 

This architecture separates the revenue distribution logic from the assets themselves, allowing the protocol to manage heterogeneous assets.

**Design Patterns**

The contracts make use of several well-known design patterns:

- Ownership - Both contracts inherit from OpenZeppelin's `Ownable` for access control
- Reentrancy protection - `LiquidInfrastructureERC20` inherits from `ReentrancyGuard`
- ERC20 and ERC721 standards - Leveraging accepted token standards improves interoperability
- Allowlist - The ERC20 maintains an allowlist to restrict token holders

**Security Considerations**

**Potential Attack Vectors**:

- Attempting to acquire ERC20 rewards without allowlist approval
- Triggering out-of-gas errors to block distributions 
- Manipulating rewards distribution to take other holders' rewards

**Mitigations**:

- Allowlist gatekeeps ERC20 rewards
- Reentrancy protection prevents recursive call attacks  
- Owner abilities to restart failed distributions

**Test Coverage**

There are currently no unit tests provided in the codebase. Adding tests would significantly improve security and confidence:

- Test token locking and unlocking during distribution cycles
- Test allowlist enforcement on ERC20 transfers 
- Test owner abilities like allowlisting addresses
- Test integration of NFT revenue deposit and ERC20 distribution

**Opportunities for Decentralization** 

The ERC20 owner currently has significant control. Over time, ownership could potentially shift to a DAO. This would further decentralize control.

**Central Points of Failure**

The ERC20 owner controls critical functionality like allowlisting token holders, withdrawing from NFTs, and distributing rewards. Compromise of the owner key represents a central point of failure. 

**Adherence to Best Practices**

The contracts adhere to many best practices:

- Use of widely accepted standards like ERC20 and ERC721
- Leveraging well-tested OpenZeppelin contracts  
- Following Solidity naming conventions
- Emitting custom events for off-chain processing  

### I took a systematic approach, reviewing the codebase file-by-file, function-by-function. For each contract, I:

- Analyzed the architecture and design patterns
- Evaluated code quality 
- Identified centralization risks
- Reviewed core mechanisms
- Assessed systemic risks

I have provided specific code snippets and diagrams to illustrate the key points.

**Architecture**

The core architecture centers around two contracts:

```solidity
LiquidInfrastructureProtocol
    |
    |------------> LiquidInfrastructureERC20
    |                                    |
    |                                    |--> Ownable 
    |                                    |--> ERC20
    |                                    |--> etc...
    |
    |------------> LiquidInfrastructureNFT
                                  |
                                  |--> OwnableApprovableERC721
                                  |--> ERC721
```

* The ERC20 contract handles revenue distribution
* The NFT contract represents real-world assets
* They both inherit from OpenZeppelin contracts for access control and standards

**Code Quality**

In general, the code adheres to best practices:

```solidity
// Good use of NatSpec for documentation
/**
 * @dev Begins the Liquid Infrastructure Account recovery process [...]
*/
function recoverAccount() public onlyOwner(AccountId) {

// Follows Solidity naming conventions 
function isApprovedHolder(address account) public view returns (bool) {

// Leverages OpenZeppelin's well-tested contracts
import "@openzeppelin/contracts/access/Ownable.sol";
```

Some opportunities for improvement:

- Add comprehensive unit test coverage
- Reduce code duplication through inheritence
- Use events over comments for documentation

**Centralization Risks**

The `LiquidInfrastructureERC20` owner has significant control:

```solidity
// Owner can mint & distribute tokens
function mintAndDistribute(address account, uint amount) onlyOwner public {
   distributeToAllHolders();
   mint(account, amount);  
}

// Owner manages allowlist
function approveHolder(address holder) onlyOwner public {
   HolderAllowlist[holder] = true;
}
```

This could lead to central point of failure if owner key is compromised.

**Core Mechanisms**

The distribution mechanism locks the ERC20 during cycles: 

```solidity
// Toggle distribution lock
bool public LockedForDistribution;

// Prevent transfers during distribution
function _beforeTokenTransfer(address from, address to, uint256 amount) 
    internal virtual override {
        
    require(!LockedForDistribution, "distribution in progress");
    [...]
}

// Lock on start, unlock on end
function _beginDistribution() internal {
    LockedForDistribution = true;
}

function _endDistribution() internal {
    LockedForDistribution = false;
}
```

**Systemic Risks**  

The lack of test coverage is a systemic risk - bugs could lead to loss of funds:

```solidity
// No test folder provided
contract LiquidInfrastructureERC20 is ERC20 {
   // Many complex mechanisms
   // But no tests  
}
```

- An attacker could theoretically drain all underlying token balances to trigger a revert on `transfer()` in `distribute()`. This would halt the current distribution but not break the contract permanently.

- Attackers could also attempt to exploit the token locking mechanisms during distribution. Carefully testing all state transitions makes this unlikely.

**Recommendations** 

- Consider batching the distribution process instead of a single `distributeToAllHolders()` call

- Load test on Mainnet-fork to confirm gas usage at scale

- Ensure comprehensive test coverage of edge cases and state transitions


**The Constructor**

In the constructor, `_distributableErc20s` is simply assigned without any validation:

```solidity
constructor(
  // other params

  address[] memory _distributableErc20s
) {

  distributableERC20s = _distributableErc20s;

}
```

This means anyone could pass a malicious token contract in the array when deploying.

**setDistributableERC20s()**

This function also does a simple assignment without validation:

```solidity
function setDistributableERC20s(
  address[] memory _distributableERC20s
) public onlyOwner {

  distributableERC20s = _distributableERC20s;

}
```

So additional checks should be added in both locations.

**Mitigations**

- Maintain a whitelist of allowed tokens 

- Check `transfer()` return values in `distribute()` 

- Limit adding new distribution tokens with timelock/multisig

Adding a whitelist in the constructor and `setDistributableERC20s()` provides the best validation.



**Recommendations**

Here are some recommendations to further improve the system:

- **Add comprehensive unit test coverage** - Testing is currently lacking
- **Implement a DAO-based ownership model** - This would decentralize control away from a single key  
- **Formally verify core contract logic** - Formal verification could prove correctness of essential logic like distribution
- **Conduct security audits** - External audits may catch issues missed in internal reviews

### Time spent:
33 hours