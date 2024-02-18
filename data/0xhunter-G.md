https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#l171-l176

the loop does not break after doing its task which is to delete an array of a holder , since there are no duplicate holders the loop can break after it . 
change it to :
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
            for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                    break;
                }
            }
        }
    }