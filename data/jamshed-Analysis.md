# What is Liquid Infrastructure 

Liquid Infrastructure is a protocol to enable the tokenization and investment in real world assets which accrue revenue on-chain, and will be deployed on the Althea-L1 blockchain after launch.  
The protocol consists of tokenized real world assets represented on-chain by deployed LiquidInfrastructureNFT contracts, and the LiquidInfrastructureERC20 token that functions to aggregate and distribute revenue proportionally to holders.  
LiquidInfrastructureNFTs are flexible enough to represent devices like routers participating in Althea's pay-per-forward billing protocol, vending machines, renewable energy infrastructure, or electric car chargers.  
Liquid Infrastructure makes it possible to automatically manage these tokenized assets and arbitrarily group them, creating ERC20 tokens that represent real world assets of various classes.

Althea-L1 is a Cosmos SDK chain with an EVM compatibility layer, and the Liquid Infrastructure contracts make mention of several features that the chain will bring to Liquid Infrastructure.  
The Althea-L1 chain and its functionality provided by Cosmos SDK modules (e.g. x/bank, x/microtx, ...) are all out of scope for this audit.

 

# Approach taken in evaluating the codebase

We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.it is important to note that manual code reviews can be very time-consuming.

&nbsp;

## The main contracts are:

- LiquidInfrastructureERC20.sol
- LiquidInfrastructureNFT.sol
- OwnableApprovableERC721.sol

&nbsp;

# LiquidInfrastructureERC20.sol

`LiquidInfrastructureERC20`  is used to earn rewards from managed `LiquidInfrastructureNFTs` . Here are some key points about this contract:

1.  **Functionality**: The contract allows users to invest in real-world assets represented by `LiquidInfrastructureNFTs` and earn revenue from the network represented by the token.
    
2.  **Features**:
    
    - Minting and burning of the ERC20 token are restricted during a distribution period.
    - Revenue from managed `LiquidInfrastructureNFTs` is distributed to token holders.
    - The contract maintains a list of approved token holders and manages distributions to them.
    - It interacts with other contracts like `ERC20` , `Ownable` , `ERC721Holder` , `ERC20Burnable` , and `ReentrancyGuard` .
3.  **Events**: The contract emits various events such as `Deployment` , `DistributionStarted` , `Distribution` , `DistributionFinished` , `WithdrawalStarted` , `Withdrawal` , `WithdrawalFinished` , `AddManagedNFT` , and `ReleaseManagedNFT` .
    
4.  **Modifiers**:
    
    - `onlyOwner` : Restricts access to certain functions to the contract owner.
    - `nonReentrant` : Prevents reentrancy attacks during certain operations.
5.  **Variables**:
    
    - `ManagedNFTs` : Holds the addresses of managed `LiquidInfrastructureNFTs` .
    - `HolderAllowlist` : Maps addresses to indicate approved token holders.
    - `LastDistribution` : Records the block number of the last distribution.
    - `MinDistributionPeriod` : Specifies the minimum blocks required between distributions.
    - `LockedForDistribution` : Indicates if transfers, mints, and burns are locked during distribution.
    - `distributableERC20s` : Holds the addresses of ERC20 tokens to be distributed to holders.
6.  **Functions**:
    
    - `approveHolder` , `disapproveHolder` : Manage the list of approved token holders.
    - `distribute` , `distributeToAllHolders` : Distribute rewards to token holders.
    - `mint` , `burn` , `burnFrom` : Mint and burn tokens with distribution considerations.
    - `addManagedNFT` , `releaseManagedNFT` : Add and release managed NFTs from the contract.
    - `withdrawFromManagedNFTs` : Withdraw token balances from managed NFTs to the contract.

Overall, this contract provides a mechanism for distributing rewards from managed NFTs to token holders and ensures proper management of token holders and distributions.

# LiquidInfrastructureNFT.sol

`LiquidInfrastructureNFT` is inherited from ERC721 and `OwnableApprovableERC721` . It is designed to control a Liquid Infrastructure Account, which is a Cosmos Bank module account connected to the EVM through Althea's x/microtx module.

The contract allows for the management of thresholds for controlling the operating balances of the liquid account, the withdrawal of ERC20 balances, and the recovery process for the account.

Some key points about the contract:

- It mints an NFT token representing the account.
- It allows for setting and updating threshold values for controlling balances.
- It enables the withdrawal of ERC20 balances to the owner or a specified destination.
- It provides a recovery process for the account in case of loss.

The contract emits events for successful withdrawals, recovery attempts, successful recoveries, and changes in thresholds. Access control is implemented to ensure that only the owner or approved parties can perform certain actions within the contract.

# OwnableApprovableERC721.sol

`OwnableApprovableERC721` it serves as an abstract contract that provides two modifiers: \`onlyOwner\` and \`onlyOwnerOrApproved\` . These modifiers are derived from ERC721's \`ownerOf\` , \`getApproved\` , and \`isApprovedForAll\` functions.

Here are the key points about this contract:

- `onlyOwner(uint256 tokenId)` : This modifier ensures that a function can only be called by the owner of a specific token identified by `tokenId` .
- `onlyOwnerOrApproved(uint256 tokenId)` : This modifier allows the function to be called by either the owner of the token or by someone approved by the owner.

The contract provides a way to restrict access to certain functions based on ownership or approval status of specific tokens within an ERC721 contract. This can be useful for implementing access control in contracts where certain actions should only be performed by token owners or approved parties.

## Centralization risks

- **Owner Control**: The contract is highly centralized, with the owner having extensive powers such as minting tokens, adding or releasing managed NFTs, setting distributable ERC20s, and approving holders. This centralization introduces risks, as it relies heavily on the owner's discretion and could lead to potential abuse or mismanagement.

## Mechanism review

- **Token Distribution**: The contract implements a mechanism to distribute revenue from managed NFTs to token holders based on their ERC20 token balances. This mechanism helps incentivize token holders and aligns their interests with the revenue generated by the managed NFTs.
    
- **Locking Mechanism**: The contract includes a locking mechanism during distributions to prevent transfers, minting, and burning of tokens. This is a crucial feature to maintain the stability of token holdings during distribution periods.
    
- **Gas Limitations**: There could be gas limit issues with functions like `distributeToAllHolders` and `distribute` if there are too many holders or distributions to process in a single transaction. Implementing gas-efficient distribution strategies or batch processing could mitigate this risk.
    

## Recommendation

- **Decentralization**: Consider decentralizing control by implementing governance mechanisms that allow token holders to participate in decision-making processes, such as voting on proposals or electing representatives. This could help distribute power more evenly and increase the resilience of the contract against centralization risks.
    
- **Transparency and Accountability**: Enhance transparency by providing clear documentation on the contract's functionality, distribution mechanisms, and ownership structure. Additionally, ensure accountability by regularly communicating updates, conducting audits, and fostering community engagement.
    
- **Gas Optimization**: Optimize gas usage to prevent potential gas limit issues during distribution processes. This could involve implementing gas-efficient algorithms, batch processing of transactions, or off-chain computation where feasible.
    
- **Security Audits**: Conduct comprehensive security audits by reputable third-party firms to identify and address any vulnerabilities or weaknesses in the contract code. Regularly update the contract in response to audit findings and emerging security best practices.
    
- **Community Involvement**: Foster a vibrant and engaged community around the contract by encouraging participation, soliciting feedback, and incentivizing contributions. A strong community can enhance resilience, promote adoption, and contribute to the contract's overall success.
    

By addressing these recommendations, we can mitigate centralization risks, enhance the robustness of the contract mechanism, and build a more resilient and sustainable ecosystem for our token holders and stakeholders.

# Time spent

33 Hours

### Time spent:
33 hours