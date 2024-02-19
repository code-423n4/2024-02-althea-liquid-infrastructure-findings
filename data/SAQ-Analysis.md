## Summary

no | File |
|-|:-|
| [[File-1](#file-1)] | LiquidInfrastructureERC20.sol |
| [[File-2](#file-2)] | OwnableApprovableERC721.sol | 
| [[File-3](#file-3)] | OwnableApprovableERC721.sol | 

## Analysis Issue Report 


### [File-1]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol




#### Admin Abuse Risks:

The `approveHolder` and `disapproveHolder` functions allow the contract owner to add or remove addresses from the token holder allowlist.
There is a risk of admin abuse if the owner misuses this power to favor specific addresses or exclude others.




#### Systemic Risks:

The contract has a distribution mechanism that occurs periodically.
The `_isPastMinDistributionPeriod` function ensures that a new distribution can only start after a minimum distribution period has elapsed.
However, if the distribution process fails to complete (e.g., due to gas limits), it may leave the contract in a locked state, preventing transfers, mints, and burns until the issue is resolved.


#### Technical Risks:

1-The contract uses `OpenZeppelin` libraries, which generally have been audited and are considered secure. However, the specific version of these libraries matters, and it's crucial to use well-established and well-audited versions.

2-The contract relies on external calls to ERC20 tokens (`IERC20`), and there is a potential risk if these external calls fail or are malicious. It's essential to ensure that the external tokens are trusted and secure.


#### Integration Risks:

The contract interacts with other contracts, specifically LiquidInfrastructureNFT contracts.
The `addManagedNFT` and `releaseManagedNFT` functions allow the owner to add or release control of `LiquidInfrastructureNFTs`. Integration risks may arise if there are vulnerabilities in the interactions between this contract and the managed NFTs.


#### Non-Standard Token Risks:

The contract inherits from OpenZeppelin's `ERC20`, `ERC20Burnable`, `Ownable`, `ERC721Holder`, and `ReentrancyGuard`.
While these are widely used and well-audited, any modifications or additions should be carefully reviewed for potential vulnerabilities.

#### Overall:

while the contract has mechanisms for revenue distribution and controlled interactions with managed `NFTs`, it is essential to conduct a thorough audit, including testing edge cases and potential failure scenarios, to ensure its security and reliability.
Additionally, considering the potential complexity of tokenomics and external interactions, involving third-party security experts for an audit is highly recommended.


### [File-2]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol



#### Admin Abuse Risks:

The contract includes `OwnableApprovableERC721`, which introduces the `onlyOwner` and `onlyOwnerOrApproved` modifiers. 
These modifiers are used in functions like `setThresholds`, `withdrawBalances`, `withdrawBalancesTo`, and `recoverAccount`.
Admin abuse risks arise if the owner misuses these privileges to modify thresholds, withdraw balances improperly, or initiate unauthorized account recovery.



#### Systemic Risks:

The `recoverAccount` function initiates a recovery process that relies on the detection of a `TryRecover` event, which must be handled by the x/microtx module.
If the event is not detected or there are issues in the recovery process, it might result in systemic risks, including potential loss of funds or unintended behavior.

#### Technical Risks:

The contract relies on external calls to ERC20 tokens (`IERC20`). Any failure in these external calls may result in issues during withdrawal or recovery processes.
It is crucial to handle these external calls securely, considering potential reentrancy attacks and other vulnerabilities.


#### Integration Risks:

The contract integrates with external modules, specifically the `x/microtx` module, which plays a crucial role in account recovery.
Integration risks arise if there are compatibility issues or misalignments between this contract and external modules.


#### Non-Standard Token Risks:

The contract inherits from OpenZeppelin's ERC721 and includes an additional `OwnableApprovableERC721` contract. 
While these are widely used and well-audited, any modifications or additions should be carefully reviewed for potential vulnerabilities.


#### Overall:

the contract seems to provide functionality for controlling a Liquid Infrastructure Account, managing ERC20 thresholds, and handling recovery processes. 
However, due to the complexity of interactions and external dependencies, a comprehensive audit, including testing various scenarios, is essential to ensure the security and reliability of the smart contract. In particular, thorough testing and verification of the recovery process are crucial to mitigate potential risks. 
Additionally, involving third-party security experts for an audit is highly recommended.


### [File-3]

### The bellow issues related to the File ( Link )
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol



#### Admin Abuse Risks:

Admin abuse risks are minimal as the contract is focused on providing ownership and approval-related modifiers (`onlyOwner` and `onlyOwnerOrApproved`).
These modifiers are designed to restrict access to specific functions based on ownership or approval, reducing the potential for admin abuse.



#### Systemic Risks:

The contract relies on ERC721 functions (`ownerOf`, `getApproved`, and `isApprovedForAll`) to determine ownership and approval status. Any systemic risks would be inherited from the ERC721 standard, which is widely used and has undergone extensive testing. However, it's important to ensure that the ERC721 implementation being used is secure and reliable.


#### Technical Risks:

Technical risks are minimal in this contract.
The usage of the standard `ERC721` functions for ownership and approval checks is a standard practice, and the modifiers are straightforward. However, proper testing and verification of the ERC721 implementation being used are crucial to avoid potential technical risks.


#### Integration Risks:

The contract does not directly integrate with external modules or contracts.
It relies on the `ERC721` standard, which is a well-established standard.
Integration risks are limited to the compatibility and reliability of the ERC721 implementation used within the contract.


#### Non-Standard Token Risks:

The contract itself does not introduce non-standard tokens.
It relies on the `ERC721` standard, which is widely accepted and used in the Ethereum ecosystem.
Non-standard token risks, if any, would be associated with the ERC721 implementation being used.


#### Overall:

this contract serves as an abstraction layer providing ownership and approval-related modifiers for `ERC721-based` contracts. 
The risks associated with this contract are minimal, and its security depends largely on the security of the underlying `ERC721` implementation.
It is recommended to conduct thorough testing and use well-audited ERC721 implementations to mitigate any potential risks.


### Time spent:
2 hours