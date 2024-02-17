## Low Severity Issues Report

### L-1: DoS issues in Multiple Functions

**Lines of Code:**
- [LiquidInfrastructureERC20.sol#L421-L429](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L421-L429)
- [LiquidInfrastructureERC20.sol#L170-L177](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L170-L177)
- [LiquidInfrastructureNFT.sol#L176-L183](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L176-L183)

**Description:** Potential Denial of Service (DoS) vulnerabilities could be triggered due to unchecked external calls or interactions that may fail.

**Mitigation:** Limit arrays when you iterating over them

### Low-3: Redundant Function

**Lines of Code:**
- [LiquidInfrastructureERC20.sol#L318-L323](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L318-L323)

**Description:** The `mint` function appears redundant due to the existence of a `mintAndDistribute` function that could handle its intended functionality.

**Mitigation:** Consider removing the `mint` function if `mintAndDistribute` adequately covers all intended use cases without any loss of functionality or security.

### Low-4: No Ability to Change MinDistributionPeriod

**Lines of Code:**
- [LiquidInfrastructureERC20.sol#L75](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L75)

**Description:** The contract lacks the functionality to update the `MinDistributionPeriod`, potentially limiting flexibility in managing distribution periods.

**Mitigation:** Implement a setter function for `MinDistributionPeriod` with proper access control to allow updates by authorized parties.

### Low-5: Unnecessary Line of Code

**Lines of Code:**
- [LiquidInfrastructureERC20.sol#L431](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431)

**Description:** There exists a line of code that should be removed as it serves no purpose or is redundant.

**Mitigation:** Review and remove the unnecessary line to clean up the codebase and improve readability.

### Low-6: Redundant Entitlement Checks

**Lines of Code:**
- [LiquidInfrastructureERC20.sol#L222-L226](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L222-L226)

**Description:** Unnecessary pushing of entitlement when the value is zero, leading to inefficiencies.

**Mitigation:** Add a condition to check if the entitlement is greater than zero before pushing it to the array or executing related logic.
