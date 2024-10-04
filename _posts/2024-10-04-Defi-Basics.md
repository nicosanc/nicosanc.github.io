---
title: "Blockchain: DeFi Basics"
---

As I continue my journey into Blockchain, I want to keep track of the "Need-To-Know" concepts and terms in DeFi. My experience learning about DeFi more in depth began after a conversation with a developer at Orca. 
I use their platform as the example in a few of these notes. 

### Key DeFi Terminology and Concepts for AMMs

- **Automated Market Maker (AMM)**: A system that automates the process of providing liquidity and making markets using smart contracts.
    - **Example**: Uniswap, Orca
- **Liquidity Pool**: A collection of funds locked in a smart contract, used to facilitate trading on decentralized exchanges.
    - **Concept**: Users provide assets to the pool and earn fees from trades.
- **Liquidity Provider (LP)**: A user who supplies assets to a liquidity pool.
    - **Concept**: LPs earn trading fees in proportion to their share of the pool.
- **Impermanent Loss**: A temporary loss in value experienced by LPs when the price of their deposited assets changes compared to when they were deposited.
    - **Concept**: Occurs because LPs provide liquidity in a fixed ratio, which can lead to less returns compared to holding the assets.
- **Slippage**: The difference between the expected price of a trade and the actual price executed.
    - **Concept**: Can occur due to low liquidity or large trades impacting the pool.
- **Yield Farming**: The practice of staking or lending assets to earn rewards, often in the form of additional tokens.
    - **Example**: Providing liquidity to an AMM to earn trading fees and incentive tokens.
- **Staking**: Locking up tokens to support the network's operations, like validating transactions, and earning rewards.
    - **Example**: Staking SOL on Solana.
- **Token Swap**: Exchanging one cryptocurrency for another within a liquidity pool.
    - **Example**: Swapping USDC for SOL on Orca.
- **Governance Token** A token that gives holders voting power on protocol changes and decisions.
    - **Example**: UNI for Uniswap, ORCA for Orca.
- **DEX (Decentralized Exchange)**: An exchange that operates without a central authority, using smart contracts to facilitate trades.
    - **Example**: Orca, Uniswap

### Restaking
**Solayer**
- Take liquid stake SOL and redeposit on SOLAYER
    - **Tokenization**: When you stake liquidity in a protocol like JUP and receive a token that represents your staked position
        - EX: stJUP are unique identifiers of your specific stake on jupiter
    - **Utilization**: Solayer can accept these staked tokens as collateral or for further staking in their protocol
    - **Smart Contracts**: Solayer's smart contracts integrate with Jupiter's, ensuring that the original stake remains intact while allowing the staked coin to participate in additional protocols
    - **Yield Aggregation**: This setup enables you to earn rewards from both Jupiter and any additional activities facilitated by Solayer
- Can increase yield or provide additional security on your stake
    - **Re-Staking Rewards**: By staking the proof tokens in their protocol, Solayer offers additional rewards from their native staking or incentive programs
    - **Lending and Borrowing**: They can use the staked tokens as collateral in lending markets, generating interest or borrowing funds for yield generating activities

### Perp-Trading
**JLP**: Jupiter Liquidity Providers provide sufficient liquidity to the trading pools on their protocol:
- **Market Depth**: LPs ensure there is sufficient market depth, which reduces slippage and makes it easier for traders to execute large orders without significantly impacting the price
	- **Price Impact**: Large trades affect the price because they shift the balance of the liquidity pool. In an AMM, prices are determined by the ratio of tokens in the pool. A large trade significantly alters this ratio, causing a noticeable price change.
	- **Example**: If a large buy order consumes a significant portion of the available tokens, the price of the remaining tokens increases due to reduced supply.
	- **Slippage**: The difference between the expected price of a trade and the actual price at which the trade is executed
- **Risk**: When there's a bull market, they might not surpass holding SOL, BTC, or ETH
	- **PnL Dynamics**: Traders' loss is your gain, but traders' gain is your loss
