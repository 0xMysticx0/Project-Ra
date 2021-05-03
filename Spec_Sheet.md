> IDK how to get .md files to render LaTeX

# 1. Introduction

This is the work-in-progress ("**WIP**") spec sheet for the platform known as RA Finance (@ https://www.ra.finance/). This platform is essentially a yield aggregator / optimizer with augmented game theory features and functionalities incentivizing long term holding and benefits.

***WIP Note: Numbers will be modeled and tested before being finalized, and yes I do intend to have them be 'gamified' by approximating numbers to fibonacci / golden ratio / various conspiracy-themed numbers where possible. External docs explaining the mechanics will also be improved with flavor text according to the aesthetic theme of the project.***

# 2. Tokenomics

The reward token is the [Eye of Ra] ("**$RA**", or the "**Token**"). It has a virtually unlimited emissions schedule, but reduces algorithmically per unit of time such that it approximates a max supply. 

The Token bestows both governance and profit-sharing privileges to the holder by staking them in a smart contract (the "**Boardroom**").

## 2.1 Emission Schedule
- Approximated max supply: 7,777,777 Tokens
- [TBU] Emission schedule is expected to follow a sigmoid supply curve such that: 

$$y = \frac{1}{1 + e^{-x}}$$

## 2.2 Allocations

As the emissions schedule approximates a max supply, token allocation is possible on a fully diluted basis. A small portion of the Token will be pre-minted and allocated on day 1 while the remainder will be minted according to the emission schedule.

| Purpose | % of Max Supply | Minting Method |
|--|--|--|
| Reward Emissions | 75~88% | Emissions schedule |
| RA-TUT LGE | 5~15% | Pre-minted |
| Team Equity | 7~10% | Pre-minted / Partial vest |

# 3. Vault Overview

The core of the vault strategies are to maximize gains of underlying tokenized assets, whether they be liquidity pairs ("**LP**") or single tokens, by automatically selling reward tokens granted by various established platforms in the space into the underlying asset, thereby offering a hands-free auto-compounding service. 

## 3.1 Vault Fees

Every vault will charge fees which will apply on the underlying asset entering the vault. Conceptually, the fee is a deposit fee on the asset being staked into the vault, and also a profit fee on the profits as they are auto-compounded into the principal capital. These fees act as a deterrence against bad actor interactions with the contract while forming the basis of according an intrinsic value to the Token.
- Origination fee: [1.00]%
- Gas fee: [0.01]%

## 3.2 LP Zap

It is envisaged for the platform to offer a convenient fuss-free service to convert BNB and any other highly liquid tokens into any liquidity pair on any major DEX in a single UI without needing to venture off the platform. We may potentially use DEX aggregators in routing such cross-DEX orders to ensure minimum slippage.

Expected Input:
- Input token (BNB, BUSD, BTC, ETH, USDT, USDC, DAI, etc.)
- Liquidity pair token A
- Liquidity pair token B
- DEX LP platform

A one-click API of this function will be placed on the front-end for each vault customized to their respective underlying assets to enhance the user experience.

For the other direction of the zap, it is ideal that the API scans a list of LP tokens from the connected wallet and displays them automatically in a list. The user only then needs to decide whether to break up the LP tokens (and how much) into their constituent tokens, or to "zap" them into their desired token (such as BNB). 

In either case, an origination fee of [0.01]% is charged by the platform on the input token for a Token->LP zap. 
[***Fee structure on the reverse direction zap is pending***]

## 3.3 Multipliers

Every vault comes with a built-in multiplier feature. As emission allocations are fixed on a vault-level, the multiplier simply increases the user's share of Token emissions allocated to that vault at that point in time, where on a per block basis:

$$Tokens(user) = \frac{TVL(user)*Mult(user)}{\sum_{i=0}^n TVL(i)*Mult(i)}*totalTokenEmission$$

The multiplier starts at 1.0x (100%) on deposit. The multiplier increases by +0.1x (+10%) every 24 hours up to +9.0x (+900%) after 90 days, capping to a max multiplier of 10.0x (1,000%). 

The multiplier works on a time weighted average basis, thus withdrawals or unstaking actions do not affect the multipliers. Instead, every new deposit (whether manually by the user or automatically via autocompound) will reduce the effective displayed multiplier.

*As an example, a user has $100 of Cake-BNB held for 10 days giving a multiplier of 2x. They then deposit another new $100 of Cake-BNB. Conceptually, the old $100 LP benefits from the 2x but the new $100 LP benefits only from 1x. Mathematically this is expressed by applying a weighted average multiplier such that it now has a 1.5x multiplier on both. The frontend UI will also have a tooltip on this carefully explaining this feature to the user.* 

In other words, this multiplier works simply as a competitive TVL amplifier which further incentivizes long-term farming while adding a small gamified element to the experience.

## 3.4 APY Boost

Every vault is also outfitted with an APY Boost function. By utilizing the LP Zap function, this function automatically converts Tokens rewarded into the underlying asset, truly turning the service into a fully automatic feature requiring no supervision. 

This function is switched on by default and will require user interaction to toggle it off. When toggled off, the user will have to harvest Token emissions manually.

[**Technically possible without too much effort?**] The user will also be able to select a % of rewards to be used for the Boost, with the remaining to be untouched and subject to manual harvesting.

# 4. Governance 

The governance feature is expected to be industry standard, granting 1 voting power to each $RA token, to be applied fractionally as well. The ultimate shape of this function is expected to be modelled after https://docs.venus.io/docs/governance#introduction. In the meantime, we expect to enact a barebones aspect that is still mostly developer-driven via snapshot.org style proposals and voting for Day 1 functionality. However, smart contracts should ideally be built with variability in mind to allow for flexibility from governance votes when this feature is fully built and decentralized.

# 5. Dividends

This structure forms the basis of the intrinsic value backing the Token. Tokens staked in the Boardroom will be eligible to share in the profits generated by the platform and offers an additional use-case to the Token as another path to participate in the upside of the platform's growth.

## 5.1 Boardroom Fund

The Boardroom Fund is a smart contract / wallet that holds a portfolio of assets, including both LP tokens and single tokens. Origination fees charged by the platform are essentially sent to the Boardroom Fund. Rather than breaking up or cashing in the underlying assets, the Boardroom Fund uses these assets to generate more yield externally, which will then be converted into BNB [***BNB or BUSD, to be decided by community***].

*Example: $100,000 of Cake-BNB PCS-LP is deposited into the platform. The platform charges 1% origination fee, which is $1,000 of Cake-BN PCS-LP. This LP is sent to the Dividend Fund, where it is staked in PCS. Assuming a daily APR of 0.20%, the $2 worth of Cake is generated from this LP daily  which is converted into $2 worth of BNB. This amount is then distributed to the appropriate Strategic Fund and the Dividend Pool.* 

BNB generated this way is ring-fenced and are partially allocated to a fund that will be used to expand and develop the project (the "**Strategic Fund**"), and mostly allocated to a pool that will payout dividends to Boardoom stakers (the "**Dividend Pool**").

| Allocation | % of Yield Generated by Boardroom Fund |
|--|--|
| Strategic Fund | [30]% |
| Dividend Pool | [70]% |

## 5.2 Strategic Fund

The Strategic Fund will be a multi-sig wallet that acts as a project treasury for strategic expenses. Signatories are expected to consist of project leads, seed investor(s), and potentially independent directors appointed and voted for by the community.

Usage of the funds will be for various marketing and development costs, listing costs, partnerships, and any other strategic purposes. The approval of such strategic actions and expenses from the community forms the initial governance use case.

## 5.3 Dividend Pool

Distributions occur around a [weekly] cycle. Every [Sunday midnight], funds generated from the last [7 days] are collected and primed for distribution over the next [7 days], emitted equally per block.

*Example: In Week 1, $70 of funds are generated. In Week 2, $140 of funds are generated. In Week 3, $210 of funds are generated. $70 of funds will distributed equally per block throughout Week 2, $140 throughout Week 3, and $210 throughout Week 4.*

Distributions are subject to a Multiplier system similar to the Vault Multiplier system. Therefore, while the total distributions at any point in time are fixed, each user's share of the funds will depend on their share of the total pool staked adjusted by their respective multipliers.
