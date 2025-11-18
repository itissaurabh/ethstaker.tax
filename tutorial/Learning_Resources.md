# Learning Resources for Ethereum Staking and ethstaker.tax

This document provides curated learning resources to help you understand the concepts necessary to work with and maintain the ethstaker.tax repository.

## Table of Contents
1. [Blockchain Fundamentals](#blockchain-fundamentals)
2. [Ethereum Basics](#ethereum-basics)
3. [Ethereum Staking](#ethereum-staking)
4. [Ethereum Technical Deep Dive](#ethereum-technical-deep-dive)
5. [Rocket Pool](#rocket-pool)
6. [MEV (Maximal Extractable Value)](#mev-maximal-extractable-value)
7. [Taxation and Accounting](#taxation-and-accounting)
8. [Developer Resources](#developer-resources)

---

## Blockchain Fundamentals

### YouTube Videos

1. **"But how does bitcoin actually work?" by 3Blue1Brown**
   - https://www.youtube.com/watch?v=bBC-nXj3Ng4
   - Duration: 26 minutes
   - Best for: Intuitive understanding of blockchain, proof-of-work, and cryptographic principles
   - Why watch: Excellent visual explanations of how blockchains work

2. **"Blockchain 101" by Anders Brownworth**
   - https://www.youtube.com/watch?v=_160oMzblY8
   - Duration: 18 minutes
   - Best for: Hands-on understanding with interactive demonstrations
   - Why watch: Shows how hashing, blocks, and chains work interactively

3. **"How does a blockchain work - Simply Explained" by Simply Explained**
   - https://www.youtube.com/watch?v=SSo_EIwHSd4
   - Duration: 6 minutes
   - Best for: Quick overview
   - Why watch: Concise introduction perfect for beginners

### Websites/Articles

1. **Blockchain Demo by Anders Brownworth**
   - https://andersbrownworth.com/blockchain/
   - Interactive tool to understand hash functions, blocks, blockchains, and distributed ledgers
   - Hands-on learning - essential for intuition

2. **"Blockchain Explained" - Investopedia**
   - https://www.investopedia.com/terms/b/blockchain.asp
   - Comprehensive written guide
   - Good reference material

---

## Ethereum Basics

### YouTube Videos

1. **"Ethereum Explained" by Finematics**
   - https://www.youtube.com/watch?v=jxLkbJozKbY
   - Duration: 8 minutes
   - Best for: High-level understanding of Ethereum
   - Why watch: Clear explanation of Ethereum's purpose and smart contracts

2. **"Ethereum in 30 minutes" by Vitalik Buterin (2014)**
   - https://www.youtube.com/watch?v=66SaEDzlmP4
   - Duration: 30 minutes
   - Best for: Understanding the original vision from Ethereum's creator
   - Why watch: Historical context and fundamental concepts

3. **"What is Ethereum?" by 99Bitcoins**
   - https://www.youtube.com/watch?v=IsXvoYeJxKA
   - Duration: 6 minutes
   - Best for: Quick introduction
   - Why watch: Simple, non-technical overview

### Websites/Articles

1. **Ethereum.org - What is Ethereum?**
   - https://ethereum.org/en/what-is-ethereum/
   - Official Ethereum Foundation resource
   - Comprehensive and authoritative

2. **"A Beginner's Guide to Ethereum" by CoinDesk**
   - https://www.coindesk.com/learn/ethereum-101/
   - Multi-part series covering Ethereum basics
   - Good for progressive learning

3. **Ethereum Whitepaper (Original)**
   - https://ethereum.org/en/whitepaper/
   - Technical but foundational
   - Read after understanding basics

---

## Ethereum Staking

### YouTube Videos

1. **"Ethereum Proof of Stake Explained" by Finematics**
   - https://www.youtube.com/watch?v=psKDXvXdr7k
   - Duration: 12 minutes
   - Best for: Understanding how Ethereum's Proof of Stake works
   - Why watch: Clear explanation of validators, attestations, and rewards

2. **"Ethereum Staking Explained" by Coin Bureau**
   - https://www.youtube.com/watch?v=UVeTvTQr4WE
   - Duration: 23 minutes
   - Best for: Practical staking guide
   - Why watch: Covers staking mechanics, risks, and rewards

3. **"How Ethereum Staking Works" by Bankless**
   - https://www.youtube.com/watch?v=tpkpW031RCI
   - Duration: 45 minutes
   - Best for: Deep technical understanding
   - Why watch: Detailed coverage of consensus mechanism

4. **"The Merge Explained" by Ethereum Foundation**
   - https://www.youtube.com/watch?v=8N10a1EBhBc
   - Duration: 8 minutes
   - Best for: Understanding Ethereum's transition to Proof of Stake
   - Why watch: Context for why staking exists

### Websites/Articles

1. **Ethereum.org - Staking**
   - https://ethereum.org/en/staking/
   - Official guide to staking
   - Comprehensive coverage of all aspects

2. **"Staking on Ethereum" - Ethereum Foundation**
   - https://ethereum.org/en/staking/solo/
   - Solo staking guide
   - Technical requirements and setup

3. **"Understanding Ethereum Staking Rewards" by Attestant**
   - https://www.attestant.io/posts/understanding-ethereum-staking-rewards/
   - Detailed breakdown of reward mechanisms
   - Essential for understanding this repository's calculations

4. **ETH2 Book by Ben Edgington**
   - https://eth2book.info/
   - Comprehensive technical resource
   - Deep dive into Ethereum's consensus layer

---

## Ethereum Technical Deep Dive

### YouTube Videos

1. **"Ethereum Protocol Explained" by ETHGlobal**
   - Search on YouTube for "ETHGlobal Ethereum Protocol"
   - Various workshop videos
   - Best for: Developer-focused understanding

2. **"How Ethereum Works" by Computerphile**
   - https://www.youtube.com/watch?v=Wgh9XyjxNsE
   - Duration: 16 minutes
   - Best for: Technical audience
   - Why watch: Computer science perspective on Ethereum

### Websites/Articles

1. **Ethereum Yellow Paper**
   - https://ethereum.github.io/yellowpaper/paper.pdf
   - Formal specification of Ethereum protocol
   - For deep technical understanding (advanced)

2. **"Ethereum EVM Illustrated" by TakenobuT**
   - https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf
   - Visual guide to Ethereum Virtual Machine
   - Helpful for understanding smart contracts

3. **Ethresear.ch**
   - https://ethresear.ch/
   - Ethereum research forum
   - Cutting-edge discussions on protocol development

4. **Ethereum Improvement Proposals (EIPs)**
   - https://eips.ethereum.org/
   - All proposals for Ethereum changes
   - Reference for understanding upgrades

### Key EIPs for this Repository

1. **EIP-4895: Beacon chain push withdrawals**
   - https://eips.ethereum.org/EIPS/eip-4895
   - Enabled withdrawals post-Shanghai
   - Critical for understanding withdrawal indexing

2. **EIP-7251: Increase MAX_EFFECTIVE_BALANCE**
   - https://eips.ethereum.org/EIPS/eip-7251
   - Enables validator consolidation
   - Future feature (Issue #115)

3. **EIP-1559: Fee market change**
   - https://eips.ethereum.org/EIPS/eip-1559
   - Base fee + priority fee mechanism
   - Important for understanding block rewards

---

## Rocket Pool

### YouTube Videos

1. **"What is Rocket Pool?" by Rocket Pool**
   - https://www.youtube.com/watch?v=b-j8a-00ozQ
   - Duration: 3 minutes
   - Best for: Quick introduction to Rocket Pool
   - Why watch: Official overview

2. **"Rocket Pool Staking Explained" by Finematics**
   - https://www.youtube.com/watch?v=yANDfKVXeXk
   - Duration: 10 minutes
   - Best for: Understanding how Rocket Pool works
   - Why watch: Clear explanation of node operators and rETH

3. **"Rocket Pool Deep Dive" by Bankless**
   - Search YouTube for "Bankless Rocket Pool"
   - Duration: 60+ minutes
   - Best for: Comprehensive understanding
   - Why watch: Detailed technical and economic discussion

### Websites/Articles

1. **Rocket Pool Documentation**
   - https://docs.rocketpool.net/
   - Official documentation
   - Essential reference for understanding minipool mechanics

2. **"Node Operator Guide" by Rocket Pool**
   - https://docs.rocketpool.net/guides/node/
   - Step-by-step guide
   - Explains bond amounts, fee commissions, and rewards

3. **Rocket Pool Smart Contracts**
   - https://github.com/rocket-pool/rocketpool
   - Source code
   - For understanding fee distributor logic

4. **"Understanding Rocket Pool Node Rewards"**
   - https://docs.rocketpool.net/guides/node/rewards.html
   - Explains RPL rewards and smoothing pool
   - Critical for understanding `rewards.py` Rocket Pool logic

---

## MEV (Maximal Extractable Value)

### YouTube Videos

1. **"What is MEV?" by Finematics**
   - https://www.youtube.com/watch?v=6jfQi0X4mrY
   - Duration: 11 minutes
   - Best for: Introduction to MEV
   - Why watch: Explains front-running, back-running, and sandwich attacks

2. **"MEV Explained" by Flashbots**
   - Search YouTube for "Flashbots MEV"
   - Various educational content
   - Best for: Understanding MEV-Boost and relays
   - Why watch: From the leading MEV research organization

3. **"Ethereum MEV Deep Dive" by The Defiant**
   - Search YouTube for "The Defiant MEV"
   - Duration: 30+ minutes
   - Best for: Comprehensive coverage
   - Why watch: Interviews with researchers and builders

### Websites/Articles

1. **"Ethereum is a Dark Forest" by Dan Robinson**
   - https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest
   - Foundational MEV article
   - Eye-opening introduction to MEV

2. **Flashbots Documentation**
   - https://docs.flashbots.net/
   - MEV-Boost documentation
   - Essential for understanding MEV detection in this repo

3. **"MEV-Boost Explained"**
   - https://boost.flashbots.net/
   - How validators use MEV relays
   - Important for understanding why block rewards vary

4. **MEV Relays**
   - List of relays: https://www.mevboost.org/
   - Understanding different relay providers
   - Relevant for MEV detection code

---

## Taxation and Accounting

### YouTube Videos

1. **"Cryptocurrency Tax Explained" by CoinDesk**
   - Search YouTube for "CoinDesk crypto tax"
   - Duration: 10-15 minutes
   - Best for: Basic tax concepts
   - Why watch: Understanding why tax tracking matters

2. **"Staking Taxes Explained"**
   - Search YouTube for "crypto staking taxes"
   - Various creators cover this topic
   - Best for: Understanding income vs capital gains
   - Why watch: Context for what this tool calculates

### Websites/Articles

1. **"Cryptocurrency Tax Guide" by CoinTracker**
   - https://www.cointracker.io/blog/cryptocurrency-tax-guide
   - Comprehensive tax guide
   - Covers different tax events

2. **"Staking Rewards and Taxes" by TaxBit**
   - https://taxbit.com/cryptocurrency-tax-guide/
   - Specific guidance on staking income
   - Important for understanding accrual vs receipt

3. **IRS Virtual Currency Guidance**
   - https://www.irs.gov/businesses/small-businesses-self-employed/virtual-currencies
   - Official US tax guidance
   - Primary source for US tax treatment

4. **"Tax Treatment of Staking Rewards" by Various Jurisdictions**
   - Research your local jurisdiction
   - Tax treatment varies by country
   - Important context for users

---

## Developer Resources

### Ethereum Development

1. **"Ethereum Development Documentation" by ethereum.org**
   - https://ethereum.org/en/developers/docs/
   - Complete developer guide
   - Covers nodes, APIs, and tools

2. **"Beacon API Documentation"**
   - https://ethereum.github.io/beacon-APIs/
   - Official beacon chain API specification
   - Essential reference for understanding `beacon_node.py`

3. **"Execution API (JSON-RPC)"**
   - https://ethereum.org/en/developers/docs/apis/json-rpc/
   - Execution layer API documentation
   - Essential for understanding `execution_node.py`

### Python and FastAPI

1. **FastAPI Documentation**
   - https://fastapi.tiangolo.com/
   - Official FastAPI docs
   - Comprehensive guide to the API framework used

2. **SQLAlchemy Documentation**
   - https://docs.sqlalchemy.org/
   - ORM documentation
   - For understanding database queries

3. **"Python Async/Await Tutorial"**
   - https://realpython.com/async-io-python/
   - Understanding asyncio
   - Critical for understanding the codebase

### PostgreSQL and Databases

1. **PostgreSQL Tutorial**
   - https://www.postgresqltutorial.com/
   - Database fundamentals
   - Helpful for understanding schema and queries

2. **"Database Indexing Explained"**
   - https://use-the-index-luke.com/
   - SQL indexing guide
   - Important for performance optimization

---

## Recommended Learning Path

### For Complete Beginners

**Week 1: Blockchain Basics**
1. Watch: "But how does bitcoin actually work?" (3Blue1Brown)
2. Play with: Anders Brownworth's Blockchain Demo
3. Read: Investopedia's Blockchain Explained
4. Tutorial: Read `01_Blockchain_Basics.md` in this folder

**Week 2: Ethereum Fundamentals**
1. Watch: "Ethereum Explained" (Finematics)
2. Read: Ethereum.org - What is Ethereum?
3. Watch: "What is Ethereum?" (99Bitcoins)
4. Tutorial: Read `02_Ethereum_Concepts.md` in this folder

**Week 3: Ethereum Staking**
1. Watch: "Ethereum Proof of Stake Explained" (Finematics)
2. Read: Ethereum.org - Staking
3. Watch: "The Merge Explained" (Ethereum Foundation)
4. Tutorial: Read `03_Staking_and_Taxation.md` in this folder

**Week 4: Technical Details**
1. Read: Ethereum.org - Developers Docs
2. Read: Beacon API Documentation (skim)
3. Watch: "How Ethereum Works" (Computerphile)
4. Tutorial: Read `04_Understanding_This_Repo.md` in this folder

### For Developers (Already Know Blockchain)

**Day 1-2: Ethereum Staking**
1. Read: Ethereum.org - Staking
2. Read: "Understanding Ethereum Staking Rewards" (Attestant)
3. Watch: "How Ethereum Staking Works" (Bankless)

**Day 3-4: APIs and Data**
1. Read: Beacon API Documentation
2. Read: Execution API Documentation
3. Explore: Beaconcha.in API

**Day 5: Rocket Pool (if relevant)**
1. Read: Rocket Pool Documentation
2. Understand: Minipool rewards distribution
3. Review: Rocket Pool smart contracts

**Day 6: MEV (if relevant)**
1. Read: "Ethereum is a Dark Forest"
2. Read: Flashbots Documentation
3. Understand: MEV detection logic

**Day 7: Repository Deep Dive**
1. Read: REPOSITORY_DOCUMENTATION.md
2. Review: Database schema (`src/db/tables.py`)
3. Trace: API endpoint (`src/api/api_v2/endpoints/rewards.py`)
4. Run: Local instance and explore

### For Tax Professionals

**Focus Areas**:
1. Watch: "Cryptocurrency Tax Explained" (CoinDesk)
2. Read: IRS Virtual Currency Guidance (or your jurisdiction)
3. Read: "Staking Rewards and Taxes"
4. Tutorial: Read `03_Staking_and_Taxation.md` sections on tax treatment
5. Review: Sample tax reports from the tool
6. Understand: Accrual vs receipt accounting

---

## Additional Resources

### Podcasts

1. **Bankless Podcast**
   - http://podcast.banklesshq.com/
   - Weekly Ethereum news and deep dives
   - Great for staying current

2. **Unchained Podcast**
   - https://unchainedpodcast.com/
   - Interviews with industry leaders
   - Broader crypto context

3. **The Daily Gwei**
   - https://thedailygwei.substack.com/
   - Daily Ethereum news
   - Quick updates

### Community Forums

1. **r/ethstaker (Reddit)**
   - https://www.reddit.com/r/ethstaker/
   - Community of Ethereum stakers
   - Practical staking discussions

2. **Ethstaker Discord**
   - https://discord.io/ethstaker
   - Active community support
   - Real-time help

3. **Ethereum Magicians**
   - https://ethereum-magicians.org/
   - Technical Ethereum discussions
   - EIP discussions

### Tools for Learning

1. **Etherscan**
   - https://etherscan.io/
   - Explore Ethereum blockchain
   - See real transactions, blocks, and contracts

2. **Beaconcha.in**
   - https://beaconcha.in/
   - Explore beacon chain
   - See validators, attestations, and proposals

3. **Rocketscan**
   - https://rocketscan.io/
   - Rocket Pool explorer
   - See minipools and node operators

4. **Rated.Network**
   - https://www.rated.network/
   - Validator performance metrics
   - Understanding validator effectiveness

---

## Glossary of Key Terms

Quick reference for common terms (full explanations in tutorial files):

- **Validator**: An entity that participates in Ethereum consensus by proposing and attesting to blocks
- **Attestation**: A validator's vote on the current state of the chain
- **Epoch**: 32 slots (~6.4 minutes)
- **Slot**: 12-second period where a block can be proposed
- **Finality**: Point after which blocks cannot be reverted
- **MEV**: Maximal Extractable Value - profit from transaction ordering
- **Consensus Layer (CL)**: The beacon chain, handles proof-of-stake consensus
- **Execution Layer (EL)**: The EVM chain, handles transactions and smart contracts
- **Withdrawal**: ETH removed from a validator (partial or full)
- **Rocket Pool**: Decentralized staking protocol
- **Minipool**: Rocket Pool's smart contract representing a single validator
- **Smoothing Pool**: Rocket Pool feature that shares block proposal rewards
- **Fee Recipient**: Address that receives execution layer rewards from block proposals
- **Priority Fee**: Tip paid to validators to prioritize transactions
- **Base Fee**: Burned portion of transaction fees (EIP-1559)

---

## Updates and Staying Current

The Ethereum ecosystem evolves rapidly. Here's how to stay updated:

1. **Weekly Ethereum Core Dev Calls**
   - https://github.com/ethereum/pm
   - Protocol development discussions

2. **EIPs Repository**
   - https://github.com/ethereum/EIPs
   - Watch for new proposals

3. **Week in Ethereum News**
   - https://weekinethereumnews.com/
   - Weekly newsletter

4. **Ethereum Foundation Blog**
   - https://blog.ethereum.org/
   - Official announcements

---

## Contributing to Learning Resources

If you find helpful resources not listed here, please contribute:

1. Ensure the resource is high-quality and accurate
2. Add it to the appropriate section
3. Include a brief description of why it's valuable
4. Submit a pull request

---

**Note**: URLs and availability of resources may change over time. If you find broken links, please update this document.
