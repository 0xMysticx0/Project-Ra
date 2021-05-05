
> IDK how to get .md files to render LaTeX. Copy-paste the .md into https://stackedit.io/app to see LaTeX rendered.

# 1. Introduction

This is the work-in-progress ("**WIP**") spec sheet for the platform known as RA Finance (@ https://www.ra.finance/). This platform is essentially a yield aggregator / optimizer with augmented game theory features and functionalities incentivizing long term holding and benefits.

***WIP Note: Numbers will be modeled and tested before being finalized, and yes I do intend to have them be 'gamified' by approximating numbers to fibonacci / golden ratio / various conspiracy-themed numbers where possible. External docs explaining the mechanics will also be improved with flavor text according to the aesthetic theme of the project.***

# 2. Tokenomics

The reward token is the [Eye of Ra] ("**$RA**", or the "**Token**"). It has a virtually unlimited emissions schedule, but reduces algorithmically per unit of time such that it approximates a max supply. 

The Token bestows both governance and profit-sharing privileges to the holder by staking them in a smart contract (the "**Boardroom**").

## 2.1 Emission Schedule
- Approximated max supply: 7,777,777 Tokens
- Emission schedule is expected to follow a sigmoid supply curve such that: 

$$y = \frac{2*maxMint}{1 + e^{-x}}-1$$

Where:
- y = Tokens minted at any point in time
- x = years (assuming 360 days per year)
- maxMint = Max Tokens allocated for Reward Emissions

### Supply Curve / Emissions Schedule for Reward Emissions (excl. pre-minted tokens)
| Year | Tokens | % of maxMint | Minted | Y-o-y Emissions Reduction
|--|--|--|--|--|
| 0 | 0 | 0.00% | 0 | n/a
| 1 | 2,695,683 | 46.21% | +2,695,683 | n/a
| 2 | 4,442,632 | 76.16% | +1,746,949 | -35.19%
| 3 | 5,280,031 | 90.51% | +837,399 | -52.07%
| 4 | 5,623,494 | 96.40% | +343,463 | -58.98%
| 5 | 5,755,249 | 98.66% | +131,756 | -61.64%
| 6 | 5,804,485 | 99.51% | +49,236 | -62.63%
| 7 | 5,822,704 | 99.82% | +18,218 | -63.00%
| 8 | 5,829,420 | 99.93% | +6,717 | -63.13%
| 9 | 5,831,893 | 99.98% | +2,473 | -63.18%
| 10 | 5,832,803 | 99.99% | +910 | -63.20%

Emissions would follow the supply curve on a daily basis (24 hours / 28,800 blocks), with the per-block emissions being a linear average. Eg., day 1 emissions = 8,102 / 28,800 tokens per block for 24 hours, while day 200 emissions = 7,510 / 28,800 tokens per block for 24 hours.

Formula to determine emissions per 28,800 blocks:

$$z_1 = \frac{2*maxMint}{1 + e^{-\frac{x}{360}}}$$

$$z_n = \frac{2*maxMint}{1 + e^{-\frac{x}{360}}}-\frac{2*maxMint}{1 + e^{-\frac{x-1}{360}}}$$

Where:
- z = Emissions for the nth set of 28,800 blocks; $z_0$ = 0
- x = day count (assuming 28,800 block per day); $x_0$ = 0
- maxMint = Max Tokens allocated for Reward Emissions

After 10 years, we set the default case as flat emissions (ie., +910 per year), to be subject to governance voting.

## 2.2 Allocations

As the emissions schedule approximates a max supply, token allocation is possible on a fully diluted basis. A small portion of the Token will be pre-minted and allocated on day 1 while the remainder will be minted according to the emission schedule.

| Purpose | % of Max Supply | Minting Method |
|--|--|--|
| Reward Emissions | 75% | Emissions schedule |
| RA-TUT LGE | 15% | Pre-minted |
| Team Equity | 10% | Pre-minted |

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

In either case, an origination fee of [0.6]% is charged by the platform on the input token for a Token->LP zap. 

No fees will be charged for the opposite direction.

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

## 3.5 Emission Allocation

Typical farms on BSC are run by an allocation point system that is iterative on the introduction of new farms which makes it gas-costly and inefficient to rebalance emissions on a macro scale. Instead, we balance it by categories to allow for manual tweaking.

| Category | % of Emissions |
|--|--|
| Native | [50]% |
| Non-native | [45]% |
| Community Requests | [5]% |

Within each category most of the vaults will then be allocated points that are local to that category and most likely be equal at 1x. There will be potentially higher rates for partnerships and promotional periods as well.

> ### Comparable Farm Native Allocations
> 
> | Farm | Total Alloc | Pool | Alloc | % of Emissions |
> |--|--|--|--|--|
> | PCS | 54,626 | C-BNB | 4,000 | 7.32% |
> | PCS | 54,626 | Cake | 13,656 | 25.00% |
> | Auto | 2,876 | Auto-BNB | 1,500 | 52.16% |
> | Ape | 12,933 | Bnn-BNB | 4,000 | 30.93% |
> | Ape | 12,933 | Bnn-BUSD | 2,000 | 15.46% |
> | Ape | 12,933 | Banana | 3,233 | 25.00% |
> | Bakery | 1,600 | B-BNB | 30 | 1.87% |
> | Bakery | 1,600 | B-BUSD | 8 | 0.5% |
> | Bakery | 1,600 | Bake | 21 | 1.31% |
> | Swamp | 9,910 | S-BNB | 1,000 | 10.09% |
> | Swamp | 9,910 | S-BUSD | 1,000 | 10.09% |
> | Swamp | 9,910 | Swamp | 4,500 | 45.41% |
> | Goose | 20,300 | E-BNB | 6,000 | 29.56% |
> | Goose | 20,300 | E-BUSD | 6,000 | 29.56% |
> | Goose | 20,300 | Egg | [] | []% |
> | 11 | 129,580 | 11-BNB | 60,000 | 46.30% |
> | 11 | 129,580 | 11 | [] | []% |
> 
> *Note: Updated as of 5 May 2021. Some farms have additional native allocations which are excluded due to small size.*
> 
> On first glance, farms typically allocate 50-60% of emissions to native pools. However, for a few of them a significant amount is allocated to single staking which we do not need (as our single staking derives value from the Boardroom Fund). Based on the above survey, I recommend allocating 50% to natives.

# 4. Governance 

The governance feature is expected to be industry standard, granting 1 voting power to each $RA token, to be applied fractionally as well. The ultimate shape of this function is expected to be modelled after https://docs.venus.io/docs/governance#introduction. In the meantime, we expect to enact a barebones aspect that is still mostly developer-driven via snapshot.org style proposals and voting for Day 1 functionality. However, smart contracts should ideally be built with variability in mind to allow for flexibility from governance votes when this feature is fully built and decentralized.

# 5. Dividends

This structure forms the basis of the intrinsic value backing the Token. Tokens staked in the Boardroom will be eligible to share in the profits generated by the platform and offers an additional use-case to the Token as another path to participate in the upside of the platform's growth.

## 5.1 Boardroom Fund

The Boardroom Fund is a smart contract / wallet that holds a portfolio of assets, including both LP tokens and single tokens ("**Assets Under Management**" or "**AUM**"). Origination fees charged by the platform are essentially sent to the Boardroom Fund. Rather than breaking up or cashing in the underlying assets, the Boardroom Fund uses these assets to generate more yield externally, which will then be converted into BNB [***BNB or BUSD, to be decided by community***]. [***Internal: They should be programmed in a way to allow for contingency plans such as the PCS v1->v2 migration.*** ]

*Example: $100,000 of Cake-BNB PCS-LP is deposited into the platform. The platform charges 1% origination fee, which is $1,000 of Cake-BN PCS-LP. This LP is sent to the Dividend Fund, where it is staked in PCS. Assuming a daily APR of 0.20%, the $2 worth of Cake is generated from this LP daily  which is converted into $2 worth of BNB. This amount is then distributed to the appropriate Strategic Fund and the Dividend Pool.* 

BNB generated this way is ring-fenced and are partially allocated to a fund that will be used to expand and develop the project (the "**Strategic Fund**"), and mostly allocated to a pool that will payout dividends to Boardoom stakers (the "**Dividend Pool**").

| Allocation | % of Yield Generated by Boardroom Fund |
|--|--|
| Strategic Fund | [20]% |
| Dividend Pool | [80]% |

## 5.2 Strategic Fund

The Strategic Fund will be a multi-sig wallet that acts as a project treasury for strategic expenses. Signatories are expected to consist of project leads, seed investor(s), and potentially independent directors appointed and voted for by the community.

Usage of the funds will be for various marketing and development costs, listing costs, partnerships, and any other strategic purposes. The approval of such strategic actions and expenses from the community forms the initial governance use case.

## 5.3 Dividend Pool

### 5.3.1 Dividend Distributions

Distributions occur around a [weekly] cycle. Every [Sunday midnight], funds generated from the last [7 days] are collected and primed for distribution over the next [7 days], emitted equally per block.

This function is expected to be activated from [Week 3] of the launch.

*Example: In Week 1, $70 of funds are generated. In Week 2, $140 of funds are generated. In Week 3, $210 of funds are generated. $70 of funds will distributed equally per block throughout Week 2, $140 throughout Week 3, and $210 throughout Week 4.*

Distributions are subject to a Multiplier system similar to the Vault Multiplier system. Therefore, while the total distributions at any point in time are fixed, each user's share of the funds will depend on their share of the total pool staked adjusted by their respective multipliers.

### 5.3.2 Special Dividends

The Boardroom Fund will have an initial AUM:TVL ratio ("**Special Dividend Ratio**" or "**SDR**") of [1]% (in line with the origination fees). This means that the benchmark US$ value of AUM in the Boardroom Fund would be at [1]% of the TVL in the platform. This AUM:TVL ratio will also be subject to governance adjustments in the future as the platform grows.

[***Internal: To consider running this on a monthly cycle instead?*** ] The platform will calculate and display a 7-day Volume Weighted Average ("**7DVWA**") TVL as well as a 7DVWA AUM. At the end of every dividend cycle, 7DVWA-AUM exceeding the SDR of the 7DVWA-TVL will be liquidated and paid as dividends, such that the resulting 7DVWA-SDR remains at the set benchmark rate. If the SDR is below the benchmark ratio, nothing happens.

[***Any thoughts?*** ] The order of liquidation would be from lowest APY to highest APY. This method not only cycles some capital back to governance holders, but also optimizes Governance Fund APY's in the form of this limited rebalancing. Alternatively, another liquidation strategy would be to liquidate from the 5 assets that have grown the most within the consideration period, pro-rata.

This function is expected to be activated from [Week 5] of the launch.

# A. Internal Analysis (WIP)

## A.1 Pool Emission Allocations

### Top 20 TVL PCS Pools (*As of 5 May 2021*)

| # | Pool | APR | TVL (US$mm) | Alloc |
|--|--|--|--|--|
| 1 | Cake-BNB | 73.91% | 1,675 | 40x |
| 1b | Cake | 91.01% | 4,630 | []x |
| 2 | BUSD-BNB | 25.57% | 968 | 8x |
| 3 | USDT-BUSD | 13.65% | 453 | 2x |
| 4 | ETH-BNB | 13.73% | 451 | 2x |
| 5 | BTCB-BNB | 15.05% | 412 | 2x |
| 6 | USDT-BNB | 19.52% | 317 | 2x |
| 7 | BTCB-BUSD | 23.6% | 131 | 1x |
| 8 | Bunny-BNB | 26.27% | 118 | 1x |
| 9 | UNI-BNB | 32.08% | 97 | 1x |
| 10 | USDC-BUSD | 16.06% | 96 | 0.5x |
| 11 | DOT-BNB | 32.78% | 94 | 1x |
| 12 | VAI-BUSD | 17.44% | 89 | 0.5x |
| 13 | DAI-BUSD | 17.48% | 88 | 05x |
| 14 | Link-BNB | 35.98% | 86 | 1x |
| 15 | UST-BUSD | 21.72% | 71 | 0.5x |
| 16 | XRP-BNB | 47.01% | 66 | 1x |
| 17 | XVS-BNB | 58.22% | 53 | 1x |
| 18 | ALPHA-BNB | 65.36% | 47 | 1x |
| 19 | ADA-BNB | 32.69% | 47 | 0.5x |
| 20 | Alpaca-BNB | 71.34% | 43 | 1x |

Roughly $10bn TVL total ($3.7bn excl. Cake & Cake-BNB). If we aim to chip off 2% of their non-native market share, that would be c.$74mm TVL, with roughly 20% APR (0.055% APRD) for the big chip pairs. Incentivizing these TVL to switch would require a) relatively quick origination fee payback, and b) superior stabilized APY's, ceteris paribus. If we assume that the use of yield agg's would mostly resolve (b), then native emission allocations would have to solve (a). Realistically, we would not blindly take the top 20 TVL pairs, and instead focus on the ones where we can pragmatically take TVL from. Analysis of other PCS/Goose clones have indicated that the most liquid pairs tend to be centered around the following big chips: BNB, BUSD, BTCB, ETH, DAI, USDC, DOT, CAKE. It is likely that we will focus on these farms as a start, in combination with our UFO-recommended projects such as NRV and BGOV.

## A.2 Valuation

[tbd]

# B. Future Development (WIP)

## B.1 Farm Collateral

Where we allow vaults to be enabled as collateral. We can even offer two options when users enable their farm as collateral: a) minting a RA-native stablecoin, and b) leveraged yield farming. 

Precedents for leveraged yield farming: Alpha, Alpaca, Bigfoot
Precedents for lending protocols: Venus, Fortress
Precedents for farm collateral: RampDefi

## B.2 Specialized Cake Strategies

### B.2.1 Syrup Optimizer Vault

This vault iterates through all Syrup pools, and constantly finds the highest APR pool to stake in, converting the relevant syrup reward token back into Cake. Depending on gas and any other hidden fees, structure may change such as switching based on an average APR, or using an allocation of top 3 APR pools instead.

### B.2.2 PCB Vault

Pancakebunny currently has the best Cake APR, partly due to their fee structure supporting the price floor of Bunny. However, their 0.5% withdrawal fee (if withdrawn within 72 hours) makes autocompounding structures tricky, along with their abnormally high gas fees for harvesting. To check ValueDefi's Cake strategies to see how they deal and offset with this.
