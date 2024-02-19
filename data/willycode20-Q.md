[L-1] The public function distributeToAllHolders() in LiduidInfrastructure.sol contract should be internal

mitigation: This is to prevent users from calling it, thereby allowing them spend unintended gas fees

[L-2] The public function distributeToAllHolders() in LiquidInfrastructure.sol is missing access control modifier.

mitigation: A malicious user can call this function attempting to send distribution to all holders. 

[N-1] function withdrawFromAllMangedNFTs() in LiquidInfrastructure.sol is missing developer comment

mitigation: consider putting comments for easier readability and auditability

