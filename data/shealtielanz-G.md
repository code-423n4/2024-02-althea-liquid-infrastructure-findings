`Gas Optimizations.`
# Report
## Gas Saving Findings 
**Gas Findings List**
| Number | Issue Details | Instances |
|-----:|----|-----|
| G-01 | Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs. | 1 |
| G-02 | Use modifiers instead of repeating checks in different functions. | 3 |
| G-03 | Delete the `mint()` function to save deployment gas | 1 |
| G-04 | Merge the `burnAndDistribute` & `burnFromAndDistribute` functions using a bool | 1 |


# G-01 Reconstuct the `addManagedNFT()` to save gas by calling the function once to add multiple NFTs.
## Summary.

## Recommendation.


# G-03 Delete the `mint()` function to save deployment gas.
## Summary.

## Recommendation.


# G-02 Use modifiers instead of repeating checks in different functions.
## Summary.

## Recommendation.


# G-04 Merge the `burnAndDistribute` & `burnFromAndDistribute` functions using a bool
## Summary.

## Recommendation.

