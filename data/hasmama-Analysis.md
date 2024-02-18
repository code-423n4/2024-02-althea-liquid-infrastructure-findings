TABLE OF CONTENT

1. Ensuring the accuracy and security of the distribution logic.
2. Assessing Denial-of-Service (DoS) attack risk:
3. Validating the security of the distribution tokens
4. Evaluation of access control mechanisms
5. Potential for Acquiring Tokens Without Approval
6. Evaluating the access control for managing NFTs in LiquidInfrastructureERC20.sol
7. Compliance with ERC Standards:


1. Ensuring the accuracy and security of the distribution logic.

The distribution logic in LiquidInfrastructureERC20.sol is verified for correctness and security by meticulously reviewing the distribute() function, scrutinizing locking mechanisms, and validating entitlement calculations to ensure bug-free and exploit-resistant distribution processes.

Executive Summary:

Overall, the LiquidInfrastructureERC20 contract appears to have several security measures in place to protect against common vulnerabilities. However, there are a few potential areas for improvement that should be considered.

Strengths:

    The contract uses the ReentrancyGuard modifier to prevent reentrancy attacks.
    The _beforeTokenTransfer and _afterTokenTransfer functions enforce holder approval and update the holder list accordingly.
    The distribute function has safeguards to prevent distribution during locked periods and ensures that at least one distribution is processed.
    The withdrawFromManagedNFTs function has a mechanism to track progress and prevent withdrawals during distribution.

Weaknesses:

The distribute function:- does not explicitly check for the allowance of each distributableERC20 before transferring tokens. This could potentially allow an attacker to exploit an ERC20 token with a vulnerability in its transfer         function.
The addManagedNFT function:- does not verify that the contract actually owns the AccountId of the NFT before adding it to the ManagedNFTs list. This could potentially allow an attacker to add a malicious NFT to the list.
    The releaseManagedNFT function removes the released NFT from the ManagedNFTs list by iterating over the list and overwriting elements. This approach could be inefficient for large lists.

Recommendations:

    Explicitly check the allowance of each distributableERC20 before transferring tokens in the distribute function.
    Verify that the contract owns the AccountId of the NFT before adding it to the ManagedNFTs list in the addManagedNFT function.
    Consider using a more efficient data structure for managing the ManagedNFTs list, such as a mapping.



2. Assessing Denial-of-Service (DoS) attack risk:

Executive Summary:

The review involved manual analysis of the source code and testing its functionality.

The LiquidInfrastructureERC20 contract has several potential Denial-of-Service (DoS) vulnerabilities. The most critical one is related to the distributeToAllHolders function, which iterates through all token holders and distributes tokens to them. This function could be exploited by an attacker with a large number of controlled accounts to exhaust gas and prevent legitimate distributions.

Detailed Findings:

DoS vulnerabilities:

distributeToAllHolders:- This function iterates through all token holders and distributes tokens to them. An attacker with a large number of controlled accounts could call this function and exhaust gas, preventing legitimate distributions.
withdrawFromManagedNFTs:- This function iterates through a list of managed NFTs and withdraws tokens from them. An attacker could add a large number of malicious NFTs to the list and exploit this function to exhaust gas.

Other vulnerabilities:

    Lack of access control for addManagedNFT:- Anyone can call the addManagedNFT function to add an NFT to the list of managed NFTs. This could be exploited by an attacker to add malicious NFTs that could steal tokens or perform other 	harmful actions.
    Potential reentrancy vulnerability in withdrawFromManagedNFTs:- The withdrawFromManagedNFTs function does not appear to have any reentrancy protection. This could be exploited by an attacker to reenter the function and steal tokens.

Recommendations:

    Implement a cap on the number of distributions that can be performed in a single transaction.
    Consider using a more efficient algorithm for distributing tokens, such as batching transactions.
    Implement access control for the addManagedNFT function.
    Add reentrancy protection to the withdrawFromManagedNFTs function.





3. Validating the security of the distribution tokens


Excutive Summary

This report reviews the security of token distribution in the LiquidInfrastructureERC20.sol contract. The contract is designed to withdraw revenue from managed LiquidInfrastructureNFTs and distribute it proportionally to token holders. Key functions under review include distributeToAllHolders, distribute, setDistributableERC20s, and the constructor.

3.1 Methodology:

The review involved manual analysis of the source code and testing its functionality.

3.2 Findings:

3.a Potential Manipulation of _distributableERC20s:

    The setDistributableERC20s function allows the owner to update the list of distributable ERC20 tokens without any validation. An attacker could exploit this function to add malicious tokens, potentially stealing funds from holders during distribution.

3.b Lack of Access Control for Adding Managed NFTs:

    Anyone can call the addManagedNFT function to add an NFT to the list of managed NFTs. This could be exploited by an attacker to add malicious NFTs that could steal tokens or perform other harmful actions during withdrawals.

3.c Reentrancy Vulnerability in withdrawFromManagedNFTs:

    The withdrawFromManagedNFTs function does not appear to have any reentrancy protection. This could be exploited by an attacker to reenter the function and steal tokens during withdrawals.

3.d Gas Exhaustion in distributeToAllHolders:

    The distributeToAllHolders function iterates through all token holders and distributes tokens to them. An attacker with a large number of controlled accounts could call this function and exhaust gas, preventing legitimate distributions.

3.3 Recommendations:

    Implement access control mechanisms for both setDistributableERC20s and addManagedNFT. Only authorized entities should be able to modify these lists.
    Add reentrancy protection to the withdrawFromManagedNFTs function.
    Consider batching transactions or implementing other optimizations to mitigate potential gas exhaustion attacks in distributeToAllHolders.
    Validate the addresses of _distributableErc20s in the constructor to prevent manipulation.





4. Evaluation of access control mechanisms

Excutive Summary

This report evaluates the access control mechanisms implemented in the LiquidInfrastructureERC20.sol contract, focusing on approving/disapproving token holders, enforcing approval checks during transfers, and the LiquidInfrastructureNFT.sol access controls.

4.1 Methodology:

The review involved manual analysis of the source code and testing its functionality.

4.2 Findings:

4.a Approving/Disapproving Token Holders:

    The approveHolder and disapproveHolder functions are correctly implemented and restrict holder approval to the contract owner.
    However, there is no mechanism to prevent the owner from accidentally or maliciously disapproving all holders, effectively halting distributions.

4.b Enforcing Approval Checks:

    The _beforeTokenTransfer hook effectively enforces the holder approval check for receiving addresses.
    However, the check is not applied to the sender address, potentially allowing unapproved holders to transfer tokens to approved holders (though they cannot hold them themselves).

4.c LiquidInfrastructureNFT.sol Access Controls:

    This report assumes the LiquidInfrastructureNFT.sol contract has its own security considerations and is not reviewed here.
    It is crucial to ensure that the NFT contract implements proper access controls for managing and withdrawing funds from the NFTs to prevent unauthorized actions.

4.3 Recommendations:

    Implement a mechanism to prevent the owner from accidentally disapproving all holders, such as requiring a multi-signature approval process.
    Extend the holder approval check in _beforeTokenTransfer to the sender address as well.



5. Potential for Acquiring Tokens Without Approval

Excutive Summary

This report assesses the potential for acquiring ERC20 tokens without approval in the LiquidInfrastructureERC20.sol contract. It focuses on vulnerabilities or flaws that could allow unauthorized parties to:

    Obtain ERC20 tokens without being approved holders.
    Claim rewards without meeting approval requirements.
    Exploit the token minting process for unauthorized token acquisition.

5.1 Methodology:

The analysis involved manual code review and testing of the contract's functionalities.

5.2 Findings:

5a. Acquiring Tokens Without Approval:

    The _beforeTokenTransfer hook enforces the approval check for receiving addresses, preventing unauthorized transfers to unapproved holders.
    However, the check is not applied to the sender address, potentially allowing unapproved holders to transfer tokens to approved holders (though they cannot hold them themselves). This is a vulnerability that could be exploited for money laundering or other malicious purposes.

5b. Claiming Rewards Without Approval:

    The distribute function requires the contract to be in the LockedForDistribution state, which is triggered by the owner.
    Only approved holders are eligible to receive rewards during distribution.
    There are no identified vulnerabilities that allow unauthorized parties to claim rewards without approval.

5c. Token Minting and Unauthorized Acquisition:

    The mint function is restricted to the contract owner, preventing unauthorized minting.
    Minting triggers distribution if the minimum distribution period has passed, but only approved holders receive rewards.
    There are no identified vulnerabilities in the minting process that could lead to unauthorized token acquisition.

5.3 Recommendations:

    Apply the approval check to both sender and receiver addresses in _beforeTokenTransfer to prevent unauthorized transfers.
    Consider implementing a Know Your Customer (KYC) or whitelist approach to further restrict token holding and reward eligibility.




6. Evaluating the access control for managing NFTs in LiquidInfrastructureERC20.sol

Executive Summary:

This report evaluates the access control mechanisms for managing NFTs in LiquidInfrastructureERC20.sol, focusing on preventing unauthorized parties from draining funds. While some security measures are implemented, potential vulnerabilities exist and improvements are recommended.

6.1 Access Control Mechanisms:

	6a. Adding Managed NFTs:-   Restricted to owner (onlyOwner), requires contract ownership of the added NFT. Emits AddManagedNFT event for transparency 

	6b. Removing Managed NFTs:- Restricted to owner (onlyOwner) Uses manual removal loop, susceptible to potential errors or omissions, requires additional verification after removalEmits ReleaseManagedNFT event

6.2 Vulnerability Considerations:

    Manual removal loop: The manual loop for removing Managed NFTs increases the risk of errors or omissions, potentially leaving vulnerable NFTs in the array.
    Authorization checks: While owner-only access is enforced, consider implementing role-based access control (RBAC) for granular permissions, especially in multi-owner scenarios.
    Reentrancy: The withdrawFromManagedNFTs function might be susceptible to reentrancy attacks if external calls within the loop are not protected.

6.3 Recommendations:

    Replace manual loop: Implement a safer removal mechanism, like iterating backwards in the array or using dedicated data structures for efficient and secure removal.
    Consider RBAC: Evaluate the benefits of RBAC for more granular access control over NFT management, especially for larger teams or organizations.
    Protect against reentrancy: Ensure proper reentrancy protection for functions interacting with external contracts, potentially using ReentrancyGuard or similar mechanisms.



7. Compliance with ERC Standards:

    ERC20:
        The contract inherits from ERC20 and seems to implement most of its required functions like transfer, balanceOf, and totalSupply.
        However, the _beforeTokenTransfer function restricts transfers during distributions and mint/burn when the minimum distribution period hasn't passed, which deviates from the standard behavior.
    ERC721:
        The contract doesn't directly interact with ERC721 tokens. It inherits from ERC721Holder which allows holding ERC721 tokens but doesn't implement the full standard.

Additional Requirements:

    Minting and managing a single LiquidInfrastructureNFT with ID 1:
        The contract doesn't seem to have specific functionality for minting or managing NFTs directly.
        The addManagedNFT function allows adding existing NFTs to the ManagedNFTs array, but there's no mention of creating a specific NFT with ID 1.
    Distributing revenue from managed NFTs:
        The withdrawFromAllManagedNFTs function seems to handle withdrawing ERC20 tokens from managed NFTs, but it's unclear how the revenue is distributed to holders.
        The distributeToAllHolders and distribute functions distribute ERC20 tokens based on specific conditions, but it's not clear if the source of these tokens is the withdrawn revenue from managed NFTs.

Recommendation:

    Address ERC20 deviations: Clearly document the intended behavior of transfer restrictions and explain why they are necessary. Consider offering an alternative mechanism for transfers during distributions if possible.
    Clarify NFT management: If managing a specific NFT with ID 1 is a requirement, implement functionality for its creation and management.
    Connect revenue distribution: Explicitly link the withdrawn revenue from managed NFTs to the distributed ERC20 tokens. Ensure the distribution mechanism aligns with your intended rules and complies with relevant regulations.











### Time spent:
49 hours