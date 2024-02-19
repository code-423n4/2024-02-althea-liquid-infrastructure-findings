### Advanced Analysis Report for [Althea-Liquid-Infrastructure](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure) by K42

#### Overview
- [Althea-Liquid-Infrastructure](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure) introduces a multi-contract model to create unique interactions between ``ERC20`` and ``ERC721`` tokens, underpinning asset distribution and management in a DeFi context. It leverages ``NFTs`` as asset containers, linking fungible token distributions to specific ``NFT`` characteristics.

#### Understanding the Ecosystem:
- [LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol): The primary distribution logic contract, facilitating ``ERC20`` token distribution linked to ``NFT`` holdings. Distributions, entitlements, and holder management are central features.
- [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol): A specialized ``ERC721`` contract controlling access to ``ERC20`` withdrawal functionality. Owns ``ERC20s``, enabling automated management through internal balances and thresholds.
- [OwnableApprovableERC721.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol): Extends ``ERC721`` to provide a nuanced ownership, approval, and access control mechanism for ``NFTs``.

#### Codebase Quality Analysis:

**1. Key Data Structures and Libraries**
- The contracts utilize standard ``OpenZeppelin`` libraries for ``ERC20`` and ``ERC721`` functionality, ensuring robustness and security. Custom data structures include mappings for holder allowances and entitlements, which are critical for managing distributions and access controls efficiently.

**2. Use of Modifiers and Access Control**
- Access control is managed through Ownable and custom ``onlyOwnerOrApproved`` modifiers, ensuring actions can only be performed by authorized entities. The ``nonReentrant`` modifier is used to prevent re-entrancy attacks, a critical security measure for contracts handling token transfers.

**3. Use of Key State Variables**
- ``distributeToAllHolders``: The ``LockedForDistribution`` boolean and state variables related to entitlements and holder tracking control flow in ``distributeToAllHolders``. Review state transitions triggered by external actions for unintended access patterns.

**4. Use of Events and Logging**
- Essential Events: Events like ``DistributionStarted``, ``Distribution``, and ``Withdrawal`` indicate critical state changes.
- Enhancements: Consider logging additional events during contract initialization, distribution phases, and ``NFT`` operations to boost auditability and debugging capabilities.

**5. Key Functions that need special attention**
- ``distributeToAllHolders``, ``distribute`` (LiquidInfrastructureERC20): Iterations over a potentially large pool of holders warrant specific attention:
**Gas Optimization:** Introduce pagination or checkpoints to split up larger holder sets into smaller portions during distribution operations, saving gas costs.
**Merkle Trees:** For very large distributions, Merkle tree-based approaches allow more efficient claiming mechanisms if appropriate for the system.
- ``withdrawBalances``, ``withdrawBalancesTo`` (LiquidInfrastructureNFT): Withdrawal management is a critical part of the contract's functionality:
**Authorization Checks:** Thoroughly examine logic in these functions to confirm sufficient checks against thresholds to mitigate risks of excessive or unauthorized withdrawals.
- ``setThresholds`` (LiquidInfrastructureNFT):
**Validation:** Verify the accuracy and integrity of input data supplied to this function, avoiding incorrect withdrawal limit configurations.

**6. Upgradability**
- The contracts do not currently implement an upgradability pattern. Considering upgradability could ensure long-term flexibility and adaptability of the infrastructure.

#### Architecture Recommendations:
- Implement a modular design to enhance maintainability and scalability. Consider separating concerns by using distinct contracts for different functionalities, connected through interfaces.
- Evaluate the use of proxy contracts for upgradability, allowing for future improvements without losing state or compromising security.

#### Centralization Risks:
- The heavy reliance on the `onlyOwner` modifier in critical functions introduces centralization risks. Explore mechanisms for decentralized governance, such as DAOs or multisig wallets, to distribute control.
- The ``HolderAllowlist`` mechanism, while providing a layer of security, centralizes the power to approve or disapprove holders. Consider a decentralized approval process or community governance for holder management.

#### Mechanism Review:
- The distribution mechanism in ``LiquidInfrastructureERC20`` could benefit from optimization to handle large numbers of holders more efficiently. Investigate the feasibility of using off-chain computations with on-chain verification to reduce gas costs.
- The withdrawal mechanism in ``LiquidInfrastructureNFT`` should incorporate checks against the current balance of ``ERC20`` tokens to prevent over-withdrawals or errors during execution.

#### Systemic Risks:
- The interconnectedness of ``ERC20`` distributions and ``ERC721`` access control introduces systemic risks, particularly if one contract behaves unexpectedly. Implement comprehensive testing and scenario analysis to identify potential failure points.
- Dependency on external ``ERC20`` tokens for distributions could expose the system to risks if those tokens have vulnerabilities. Conduct thorough audits of all integrated tokens.

#### Areas of Concern in specific functions in each contract:
- The `distributeToAllHolders` function in ``LiquidInfrastructureERC20`` and the `withdrawBalancesTo` function in ``LiquidInfrastructureNFT`` are particularly sensitive to gas optimization and authorization checks. Rigorous testing and validation are recommended to ensure these functions operate securely and efficiently under all conditions.

#### Codebase Analysis:
## [LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol) Contract
- Overview: A contract for `ERC20` token distribution linked to `NFT` holdings, incorporating `ERC20`, `ERC20Burnable`, `Ownable`, `ERC721Holder`, and `ReentrancyGuard` features for secure and efficient distribution within a DeFi ecosystem.

**Key Observations and Recommendations:**
- Event Utilization: Ensure comprehensive event logging for transparency and auditability.
- Distribution Mechanism: Implement gas optimization strategies like pagination or checkpoints, and consider Merkle trees for large-scale distributions.
- Access Control: Review access control logic for intended use cases and security requirements. Consider decentralized governance.
- Token Transfer Restrictions: Validate restrictions on token liquidity and user experience.
- Minting and Burning: Ensure minting and burning actions are appropriately gated.
- Withdrawal Management: Validate withdrawal logic to prevent unauthorized access or manipulation.
- Upgradability and Flexibility: Consider adopting a proxy-based upgradability pattern.
- Centralization Concerns: Evaluate decentralized governance mechanisms to mitigate risks.
- Security Practices: Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent unforeseen risks or actions.

## [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol) Contract
- Overview: Extends `ERC721`, incorporating `OwnableApprovableERC721` pattern for managing NFTs with mechanisms for setting withdrawal thresholds for `ERC20` tokens, emphasizing ownership and approval-based access control.

**Key Observations and Recommendations:**
- Event Utilization: Ensure all critical actions emit events for tracking and auditing.
- Withdrawal Mechanism: Validate the withdrawal logic rigorously to prevent unauthorized access.
- Threshold Management: Validate input arrays for consistency and integrity.
- Access Control: Review the implementation of modifiers for potential vulnerabilities.
- ``ERC20`` Interaction: Confirm ``ERC20`` tokens comply with the expected interface.
- Account Recovery: Define a clear and secure process for account recovery.
- Constructor and Initialization: Validate and sanitize the account name to prevent issues.
- Security Practices: Use [Tenderly](https://dashboard.tenderly.co/) and [Defender](defender.openzeppelin.com) for continued monitoring to prevent unforeseen risks or actions.
- Upgradability and Flexibility: Evaluate making the contract upgradable.
- Centralization Concerns: Explore more decentralized governance mechanisms.

## [OwnableApprovableERC721.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol) Contract
- Overview: An abstract contract extending the ``ERC721`` standard with ownership and approval checks tailored for specific token IDs, enhancing security and flexibility in permission management.

**Key Observations and Recommendations:**
- Modifier Implementation: Apply modifiers consistently across all functions to enforce strict access control.
- Access Control Flexibility: Consider additional roles or permissions for complex dApps or systems.
- Integration with ``ERC721``: Regularly review and test the integration with the ``ERC721`` standard.
- Error Messaging: Maintain descriptive and consistent revert messages, including token IDs where applicable.
- Use Case Specificity: Clearly document intended use cases for contracts inheriting from ``OwnableApprovableERC721``.
- Upgradability Considerations: Consider upgradability aspects for inheritors of this contract.
- Contract Extensibility: Carefully design additional features to avoid introducing security vulnerabilities.

#### Contract Details:
- Function interaction graphs I made for each contract for better visualization of function interactions:

- Link to [Graph](https://svgshare.com/i/13Hs.svg) for [LiquidInfrastructureERC20.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol).

- Link to [Graph](https://svgshare.com/i/13KR.svg) for [LiquidInfrastructureNFT.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol).

- Link to [Graph](https://svgshare.com/i/13KH.svg) for [OwnableApprovableERC721.sol](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol).

#### Conclusion:
- The [Althea-Liquid-Infrastructure](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure) presents an ecosystem for ``ERC20`` and ``ERC721`` token interaction, offering solutions for asset distribution and management. Continuous improvement and adherence to best practices in smart contract security and development will be key to mitigating future risks and ensuring the long-term success of the infrastructure.

### Time spent:
20 hours