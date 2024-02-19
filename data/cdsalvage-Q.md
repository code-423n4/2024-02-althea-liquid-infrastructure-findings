I have identified one low risk issue with the NFT contract that receives payment from the list of approved ERC-20 tokens and withdraws them to the ERC-20 contract. 

In line 179 of LiquidInfrastructureNFT.sol, the NFT contract requires the ERC-20 token to return a Boolean value indicating if the transfer succeeds. This is correct according to the ERC-20 standard, however there are many non-standard ERC-20 tokens currently deployed on ethereal main net. These non standard tokens don't return anything on the transfer function and thus would cause the withdraw method to continually fail. 

The largest stablecoin on the ethereum main net, USDT shows this behavior, among others, so these non standard ERC-20s could prove difficult to avoid. 



Proof of concept

The code for the USDT ERC-20 token is located here. the ABI shows that transfer does not return any value, thus it is clear that the code expecting the return value will fail when using this token.
https://etherscan.io/token/0xdac17f958d2ee523a2206206994597c13d831ec7#code


Remediation 

I would suggest calling transfer in line without checking for a return value, since this causes the issue above and delivers minimal utility currently. 
