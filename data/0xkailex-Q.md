## Summary
There is no check for _minDistributionPeriod value in the constructor. This parameters tends to be a special tokenomics measure to avoid distribution or mint/burn operations too often. So it must to be important to make sure the value is within expected values range.

## Impact
Low impact as setting this parameter is done only once during constructor.

## Proof of Concept
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
        ManagedNFTs = _managedNFTs;
        LastDistribution = block.number;

        for (uint i = 0; i < _approvedHolders.length; i++) {
            HolderAllowlist[_approvedHolders[i]] = true;
        }

        MinDistributionPeriod = _minDistributionPeriod;

        distributableERC20s = _distributableErc20s;

        emit Deployed();
    }
## Tools Used
Manual review

## Recommended Mitigation Steps
Recommend to check the distribution for valid ranges of value (at least non zero or ideally within some specific range)