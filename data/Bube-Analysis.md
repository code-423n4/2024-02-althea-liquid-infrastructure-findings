## Analysis Report

The scope of `Althea Liquid Infrastructure` audit consists of three smart contracts: `LiquidInfrastructuteNFT`, `LiquidInfrastructureERC20` and `OwnableApprovableERC721`. Each contract is described in details below.

## `LiquidInfrastructureNFT` contract

## The scheme of the architecture of the `LiquidInfrastructureNFT` contract:

![Imgur](https://i.imgur.com/ELPYJ05.png)

The contract inherits from `ERC721` and a custom contract `OwnableApprovableERC721`. The contract is designed to interact with Althea's x/microtx module, which is part of a Cosmos blockchain network. The contract is intended to represent a Liquid Infrastructure Account, which is an account on the Cosmos network that can receive and send microtransactions for services such as internet provision.

## Function summary

| Contract                 | Function             | Explanation                                                                              | Comments                                                                             |
| -------------------------| ---------------------| -----------------------------------------------------------------------------------------| ------------------------------------------------------------------------------------ |
| LiquidInfrastructureNFT  | getThresholds        | Returns the current threshold `ERC20` addresses and their corresponding balance amounts  | Anyone can call it                                                                   |
| LiquidInfrastructureNFT  | setThresholds        | Updates the threshold `ERC20` addresses and their corresponding balance amounts          | Only owner of the `Account Token` or approved address can call it                    |
| LiquidInfrastructureNFT  | withdrawBalances     | Withdraws all balances of specified ERC20 tokens from the contract to the owner's address| Only owner of the `Account Token` or approved address can call it                    |
| LiquidInfrastructureNFT  | withdrawBalancesTo   | Similar to `withdrawBalances`, but can set the `destination` address                     | Only owner of the `Account Token` or approved address can call it                    |
| LiquidInfrastructureNFT  | _withdrawBalancesTo  | Performs the actual transfer of `ERC20` tokens to the specified `destination`            | An internal function used by `withdrawBalances` and `withdrawBalancesTo`             |
| LiquidInfrastructureNFT  | recoverAccount       | Emits a `TryRecover` event                                                               | Only owner of the `Account Token` can call it to initiate the recovery process       |                                                                          

## Notes about `LiquidInfrastructureNFT` contract:

- The contract does not implement any rate limiting or timelocks on sensitive functions like `setThresholds` or `recoverAccount`, which could be considered depending on the use case.

## `LiquidInfrastructureERC20` contract

The contract is an `ERC20` token with additional features to manage revenue distribution from associated `NFTs` representing real-world infrastructure assets. The contract is designed to allow token holders to earn rewards from these assets automatically.

## Function summary

| Contract                   | Function                     | Explanation                                                                                       | Comments                                                                                                                              |
| ---------------------------| -----------------------------| --------------------------------------------------------------------------------------------------| ------------------------------------------------------------------------------------------------------------------------------------- |
| LiquidInfrastructureERC20  | isApprovedHolder             | Checks if an account is on the allowlist to hold tokens                                           | Only owner can call it                                                                                                                |
| LiquidInfrastructureERC20  | approveHolder                | Adds an account to the allowlist, allowing it to receive the underlying `ERC20` tokens            | Only owner can call it                                                                                                                |
| LiquidInfrastructureERC20  | disapproveHolder             | Removes an account from the allowlist, preventing it from receiving new `ERC20` tokens            | Only owner can call it                                                                                                                |
| LiquidInfrastructureERC20  | _beforeTokenTransfer         | Ensures that the contract is not locked for distribution                                          | An internal function used                                                                                                             |
| LiquidInfrastructureERC20  | _beforeMintOrBurn            | Ensures that the minimum distribution period has passed since the last distribution               | An internal function used by `_beforeTokenTransfer`                                                                                   |
| LiquidInfrastructureERC20  | _afterTokenTransfer          | Removes a holder from the holders array if their balance reaches zero                             | An internal function                                                                                                                  |
| LiquidInfrastructureERC20  | distributeToAllHolders       | Initiates a distribution to all token holders                                                     | Anyone can call it                                                                                                                    |
| LiquidInfrastructureERC20  | distribute                   | Continues or starts a distribution process, distributing rewards to a specified number of holders | Anyone can call it and has a `nonReentrant` modifier                                                                                  |
| LiquidInfrastructureERC20  | _isPastMinDistributionPeriod | Checks if the minimum distribution period has elapsed                                             | An internal function used by `_beforeMintOrBurn`, `distribute`, `mintAndDistribute`, `burnAndDistribute`, `burnFromAndDistributeOnly` |
| LiquidInfrastructureERC20  | _beginDistribution           | Prepares the contract for distribution by locking it and calculating entitlements                 | An internal function used by `distribute`                                                                                             |
| LiquidInfrastructureERC20  | _endDistribution             | Ends the distribution process and unlocks the contract                                            | An internal function used by `distribute`                                                                                             |
| LiquidInfrastructureERC20  | mintAndDistribute            | Distributes rewards and then mints new tokens                                                     | Only owner can call it                                                                                                                |
| LiquidInfrastructureERC20  | mint                         | Allows the owner to mint new tokens to a specified account                                        | Only owner can call it and has `nonReentrant` modifier                                                                                |
| LiquidInfrastructureERC20  | burnAndDistribute            | Allows a token holder to initiate a distribution and then burn their tokens                       | Anyone can call it                                                                                                                    |
| LiquidInfrastructureERC20  | burnFromAndDistribute        | Allows a token holder to initiate a distribution and then burn tokens from an approved account    | Anyone can call it                                                                                                                    |
| LiquidInfrastructureERC20  | withdrawFromAllManagedNFTs   | Initiates the withdrawal of funds from all managed NFTs                                           | Anyone can call it                                                                                                                    |
| LiquidInfrastructureERC20  | withdrawFromManagedNFTs      | Withdraws funds from a specified number of managed NFTs                                           | Anyone can call it                                                                                                                    |
| LiquidInfrastructureERC20  | addManagedNFT                | Adds a `LiquidInfrastructureNFT` contract to the managed `NFTs` list                              | Only owner can call it                                                                                                                |
| LiquidInfrastructureERC20  | releaseManagedNFT            | Releases a managed NFT to a new owner                                                             | Only owner can call it and has  `nonReentrant` modifier                                                                               |
| LiquidInfrastructureERC20  | setDistributableERC20s       | Sets the list of `ERC20` tokens that can be distributed to holders                                | Only owner can call it                                                                                                                |

## Notes about `LiquidInfrastructureERC20`:

- The contract uses `ReentrancyGuard` to prevent reentrancy attacks.

- The contract owner has significant control over the contract, including minting tokens, adding/removing NFTs, and managing the allowlist. This centralization could be a risk if the owner's account is compromised.

- The `distributeToAllHolders` function attempts to distribute rewards to all holders in a single transaction, which could run out of gas if there are too many holders. This could lead to incomplete distributions.

## `OwnableApprovableERC721` contract

This contract extends the functionality of an `ERC721` token by adding two access control modifiers: `onlyOwner` and `onlyOwnerOrApproved`. The contract is designed to be used with manually enumerated tokens, where specific token `IDs` represent different items or concepts (e.g., `RockId = 1`, `PaperId = 2`, `ScissorsId = 3`). The contract imports `Context.sol` and `ERC721.sol` from the `OpenZeppelin` library, which are standard utilities for keeping track of the message sender and implementing the `ERC721` token standard, respectively.

## Function summary

| Contract                 | Modifier             | Explanation                                                                                                                                                                                                                                                                                                      | 
| -------------------------| ---------------------| -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| 
| OwnableApprovableERC721  | onlyOwner            | Restricts the execution of a function to the owner of a `tokenId`. The modifier checks if the caller (`_msgSender()`) is the owner of the specified token `ID` by calling the `ownerOf` function from the `ERC721` standard. If the caller is not the owner, the transaction is reverted with an error message.  |
| OwnableApprovableERC721  | onlyOwnerOrApproved  | Extends the access control to not only the owner of the `tokenId` but also to an address that has been approved by the owner. This is done by calling the `_isApprovedOrOwner` internal function from the ERC721 standard.                                                                                       | 

## Approach Taken in Evaluating the Codebase

- Examination of each contract's components, including imports, state variables, functions, modifiers, and potential security flaws.
- Examination of the provided test case.
- The used framework is `hardhat`, therefore a `Foundry` setup was needed for easily testing and evaluation of the codebase.
- Read the known issues and attack ideas in the README.
- Some vulnerabilities were found and described in reports.



### Time spent:
20 hours