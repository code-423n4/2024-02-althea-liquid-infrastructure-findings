## Architecture

The system consists of LiquidInfrastructureNFT and LiquidInfrastructureERC20 contracts to enable revenue generating real-world assets to be tokenized on-chain.

- NFTs represent individual real-world revenue generating assets.
- The ERC20 aggregates multiple NFTs to allow collective investment.

This architecture provides flexibility to model different types of real-world assets. Grouping them under the ERC20 allows easy investor access.

> The core innovation that enables the Althea Liquid Infrastructure protocol is the split between the NFTs representing individual real-world revenue streams, and the ERC20 that aggregates multiple NFTs for collective investment.

**NFT Flexibility**

The NFT abstraction allows a lot of flexibility in modeling real-world assets like:

- Infrastructure devices (routers, chargers, vending machines)
- Renewable energy systems (solar, wind)
- Electric vehicles
- Real estate

Basically any asset that can generate automated on-chain revenue streams can become an NFT.

The NFT then handles thresholds for keeping operating balances, withdrawal logic, and disaster recovery features.

This provides the building blocks to tokenize diverse real-world assets.

**ERC20 Aggregation** 

The ERC20 serves as an aggregation layer on top of multiple NFTs. This provides a simple way for investors to get exposure to an entire portfolio of assets.

Rather than investing directly into each individual NFT, investors can purchase the ERC20 which automatically distributes revenue proportionally across all holders.

Some key benefits this unlocks:

- Reduced fragmentation - single ERC20 vs many NFTs
- Automated diversification 
- Lower barriers to investment
- Liquidity pooling between diverse underlyings

This dual token model delivers flexibility on the asset side and simplicity for investors.

## Design Patterns

Common patterns like Ownable, ReentrancyGuard, ERC20, ERC721 are used from OpenZeppelin.  

Custom access control modifiers check NFT ownership/approvals. This reduces code duplication.

**Use of Well-Known Libraries**

Incorporating established OpenZeppelin libraries like Ownable, ReentrancyGuard, ERC20 and ERC721 is a best practice. These building blocks have been formally audited and validate core functionality, improving security.

For example:

- Ownable centralizes permissioned control for the Owner role  
- ReentrancyGuard prevents reentrancy attacks on state changes
- ERC20 and ERC721 provide reliable, standardized token interfaces

Leveraging such well-vetted libraries reduces the need to implement and validate common functionality like transfers, balances, etc from scratch.

**Custom Access Control Modifiers**

The OwnableApprovableERC721 abstract contract demonstrates good practice by encapsulating ERC721 ownership and approval checks into custom modifiers like `onlyOwner(tokenId)` and `onlyOwnerOrApproved(tokenId)`.

These modifiers clearly express permissions, while reducing duplicate ownerOf and approval logic throughout the inheritance tree.

For example, `onlyOwnerOrApproved` explicitly requires either direct ownership or an active approval before allowing the function call to proceed.

This simplifies writing access controlled contracts that derive their policy from ERC721 ownership status.

## Security Considerations

- Access control around sensitive functions like `mint` and `withdrawals`.
- Locking of functionality during revenue distribution periods.
- Allowlist enforcement on ERC20 holders.

Main risks are around distribution logic, denial of service and unauthorized token access.

**Access Control**

Sensitive permissioned functions like `mint` and `withdrawals` are rightly restricted to just the contract owner. This follows the principle of least privilege, limiting exposure.

For example, arbitrary minting of the ERC20 could undesirably dilute existing holders. Similarly, withdrawal logic needs to be carefully controlled to avoid theft of funds.

**Locking Distribution Logic**

Locking mechanisms that block other state changing functionality during active revenue distributions help prevent potential reentrancy issues. 

For example, by restricting transfers during distribute(), it reduces the risk that a malicious receiver could attempt to recursively call back into distribution. This protects integrity of the distribution state.

**Allowlist Enforcement**  

Tightly controlling which addresses are allowed to hold ERC20 balances provides oversight over token holders. Important for compliance.

The allowlist requirement on receiving distributions provides a gatekeeper to prevent unauthorized parties from acquiring the token. This directly counters a major risk vector. 

Proper implementation of the allowlist and appropriate permissioning of the functions that manage it are therefore critical to review.

## Test Coverage

No formal test suite found in repository. Likely tests done independently. Without significant test coverage of critical functionality like distribution, withdrawals and access control lists, major issues could be missed.

## Decentralization  

The Owner role has a lot of power. Withdrawals could potentially be opened up via governance. 

Adding timelocks could decentralize control further.

The Owner currently holds significant control over critical functionality like:

- Minting new ERC20 tokens
- Managing the allowlist of approved holders
- Withdrawing funds from NFT balance sheets
- Adding/removing NFTs under contract management

This places trust in the Owner address to act appropriately. However, too much centralized control raises risks if the Owner key is compromised.

**Potential Decentralization Strategies**  

Some ways to decentralize powers away from only the Owner include:

**Governance-Controlled Withdrawals**

- Allow an elected DAO council to approve withdrawals above a threshold rather than solely the Owner. Provides oversight.

**Timelock Controls** 

- Apply a time delay to sensitive Owner functions so the community can react to potential misuse.

**Core Protocol Ownership Renunciation**
  
- Once stable, consider renouncing Ownable functionality to permanently decentralize control.

**Governance-Managed Allowlists**

- Rotate allowlist authority to governance voted addresses over time.  

Implementing such strategies through a DAO can shift powers away from the Owner to community stakeholders. This would reduce central points of failure and align with crypto ethos.

## Failure Modes

Central points of failure:
- Reliance on Owner for control of minting, burning
- No backup mechanisms if Owner key lost

**Centralized Mint/Burn Control**

The Owner possesses sole authority over minting new ERC20 tokens and burning existing tokens. 

This is risky as there are no contingencies if the Owner's private keys are lost. Without backups, no new token creation or burning could occur, disrupting economics.

Similarly, unrestricted minting powers could lead to runaway inflation if the Owner key is compromised.

**Lack of Emergency Recovery** 

There appear to be no emergency revert provisions if the Owner's keys are lost or stolen. 

Typically, timelock-gated admin functionality allows recovery as a last resort. This prevents a permanent loss of critical control.

For example, loss of Owner keys could completely halt key operations like:

- Distribution of revenue
- Adding new NFT revenue streams 
- Managing ERC20 tokenomics

Essentially bricking the core protocol without contingencies.

This single point of failure around the Owner account needs remediation to make the system resilient against disasters. 

I'd recommend instituting standard emergency mechanisms common in DeFi protocols to decentralize control and add redundancy.

## Best Practices

- Use of well-vetted libraries like OpenZeppelin.
- Code is well commented for readability.
- Follows standards like ERC20, ERC721.

## Recommendations

- Comprehensive unit test coverage, especially on distribution, withdrawals and access control.
- Timelocks and governance structures to decentralize Owner powers.
- Emergency mechanisms for account recovery/transfer if Owner key is lost. 

### Time spent:
30 hours