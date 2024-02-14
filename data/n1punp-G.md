# Set array directly to storage
The function `setThresholds` can be optimized to as shown below. Assigning the array directly to storage will automatically replace the old array with the new one.

```solidity
function setThresholds(
    address[] calldata newErc20s,
    uint256[] calldata newAmounts
) public virtual onlyOwnerOrApproved(AccountId) {
    require(
        newErc20s.length == newAmounts.length,
        "threshold values must have the same length"
    );
    thresholdErc20s = newErc20s; // directly assign array
    thresholdAmounts = newAmounts; // directly assign array
    emit ThresholdsChanged(newErc20s, newAmounts); 