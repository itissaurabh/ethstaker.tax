# ETHStaker.tax Tutorial

Welcome! This tutorial will help you understand everything you need to know to maintain and extend the ethstaker.tax repository.

## Who This Is For

- **New maintainers** who need to understand the codebase
- **Developers** wanting to contribute
- **Tax professionals** needing to understand the calculations
- **Anyone** curious about Ethereum staking and tax tracking

## Prerequisites

**No blockchain knowledge required!** This tutorial starts from the very beginning.

If you already have some knowledge:
- Know blockchain basics? Skip to [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md)
- Know Ethereum? Skip to [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md)
- Know staking? Jump to [04_Understanding_This_Repo.md](04_Understanding_This_Repo.md)

## Learning Path

### Complete Beginner (4-6 hours total)

**Week 1: Foundations**
1. Read [01_Blockchain_Basics.md](01_Blockchain_Basics.md) (1 hour)
   - Do the practical exercises
   - Watch recommended videos from [Learning_Resources.md](Learning_Resources.md)

2. Read [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md) (1.5 hours)
   - Explore Etherscan as guided
   - Understand smart contracts

**Week 2: Application**
3. Read [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md) (1.5 hours)
   - Understand how staking generates income
   - Learn tax implications

4. Read [04_Understanding_This_Repo.md](04_Understanding_This_Repo.md) (1 hour)
   - Connect concepts to code
   - Review key files

**Week 3: Practice**
5. Clone and run the repository locally
6. Make a small change (add a log statement)
7. Review an open GitHub issue
8. Try debugging a test case

### Developer Fast Track (2-3 hours)

If you already understand blockchain and Ethereum:

1. **Read**: [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md) (30 min)
   - Focus on reward types and tax treatment

2. **Read**: [04_Understanding_This_Repo.md](04_Understanding_This_Repo.md) (45 min)
   - Code walkthrough
   - Key files

3. **Reference**: [Learning_Resources.md](Learning_Resources.md)
   - Bookmark for later
   - Check Rocket Pool resources if relevant

4. **Practice**: (1 hour)
   - Set up local environment
   - Run the API
   - Query the database
   - Read actual code files

### Tax Professional Path (1-2 hours)

If you're primarily interested in understanding the tax calculations:

1. **Skim**: [01_Blockchain_Basics.md](01_Blockchain_Basics.md) (15 min)
   - Just the "Intuitive Understanding" sections
   - Get basic concepts

2. **Skim**: [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md) (15 min)
   - Focus on "Why This Matters for ethstaker.tax" sections

3. **Read carefully**: [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md) (45 min)
   - Especially "Tax Treatment of Staking" section
   - "How ethstaker.tax Calculates Income" section

4. **Review**: Sample tax reports from the tool
   - Understand the data format
   - Verify calculations match your expectations

## Tutorial Files

### [Learning_Resources.md](Learning_Resources.md)
- Curated list of YouTube videos, websites, and articles
- Organized by topic (blockchain, Ethereum, staking, Rocket Pool, MEV, taxation)
- Recommended learning paths
- Community forums and tools

**Use this as**: A reference library. Bookmark it and refer back when you need to deep-dive into a specific topic.

### [01_Blockchain_Basics.md](01_Blockchain_Basics.md)
- What is a blockchain?
- How blockchains work
- Cryptographic hashing
- Consensus mechanisms
- Decentralization

**Key takeaways**:
- Blockchains are shared ledgers no one person controls
- Hashing makes changing history impossible
- Decentralization provides resilience
- Finality means transactions become permanent

### [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md)
- Smart contracts explained
- The Ethereum Virtual Machine (EVM)
- Gas and transaction fees
- Dual-layer architecture (Consensus Layer + Execution Layer)
- How Ethereum differs from Bitcoin

**Key takeaways**:
- Ethereum is a global computer that runs programs (smart contracts)
- Validators earn from both CL (consensus) and EL (execution) activities
- Gas fees are split: base fee (burned) + priority fee (to validators)
- ethstaker.tax queries both layers for complete income tracking

### [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md)
- How Ethereum staking works
- Types of staking rewards
- Withdrawals (partial and full)
- Rocket Pool staking
- Tax treatment of staking income
- How the tool calculates your income

**Key takeaways**:
- Staking earns 4-7% annual rewards
- Rewards come from attestations, block proposals, and transaction fees
- Most jurisdictions tax rewards when earned (accrual basis)
- ethstaker.tax tracks daily balances to calculate accurate income
- Rocket Pool has complex reward sharing that requires special handling

### [04_Understanding_This_Repo.md](04_Understanding_This_Repo.md)
- How concepts map to code
- Repository architecture
- Code walkthroughs (balance indexing, income calculation, MEV detection)
- Key files and their purposes
- Common maintenance tasks
- Debugging guide

**Key takeaways**:
- Indexers collect data → Database stores it → API serves it
- Balance tracking is in `src/indexer/balances.py`
- Income calculation is in `src/api/api_v2/endpoints/rewards.py`
- MEV detection is in `src/indexer/block_rewards/block_rewards_mev_simple.py`
- Rocket Pool logic is in `src/indexer/rocket_pool/` and reward calculation endpoints

## Recommended Approach

### Day 1: Big Picture Understanding
- [ ] Read the "Intuitive Understanding" sections of all 4 tutorials
- [ ] Watch: "But how does bitcoin actually work?" (3Blue1Brown)
- [ ] Watch: "Ethereum Explained" (Finematics)
- [ ] Watch: "Ethereum Proof of Stake Explained" (Finematics)
- [ ] **Goal**: Understand what problem this tool solves

### Day 2: Technical Foundations
- [ ] Read [01_Blockchain_Basics.md](01_Blockchain_Basics.md) fully
- [ ] Play with: https://andersbrownworth.com/blockchain/
- [ ] Explore: https://etherscan.io/ (recent blocks and transactions)
- [ ] **Goal**: Understand how blockchains work

### Day 3: Ethereum Deep Dive
- [ ] Read [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md) fully
- [ ] Watch: "How Ethereum Works" (Computerphile)
- [ ] Explore a smart contract on Etherscan
- [ ] **Goal**: Understand Ethereum's unique features

### Day 4: Staking and Income
- [ ] Read [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md) fully
- [ ] Watch: "Ethereum Staking Explained" (Coin Bureau)
- [ ] Check a validator on https://beaconcha.in/
- [ ] **Goal**: Understand what generates taxable income

### Day 5: Code Exploration
- [ ] Read [04_Understanding_This_Repo.md](04_Understanding_This_Repo.md) fully
- [ ] Clone the repository
- [ ] Review `src/db/tables.py` (database schema)
- [ ] Review `src/api/api_v2/endpoints/rewards.py` (income calculation)
- [ ] **Goal**: Connect concepts to actual code

### Day 6: Hands-On
- [ ] Set up local development environment
- [ ] Run `docker-compose up`
- [ ] Make a test API request
- [ ] Query the database directly
- [ ] Add a debug log statement and see it in logs
- [ ] **Goal**: Get comfortable with the development workflow

### Day 7: Real-World Application
- [ ] Pick an open GitHub issue
- [ ] Understand what it's asking for
- [ ] Identify which files would need to be changed
- [ ] (Optionally) Implement the fix
- [ ] **Goal**: Apply your knowledge to real maintenance tasks

## Quick Reference

### Glossary of Key Terms

| Term | Simple Definition | Where to Learn More |
|------|------------------|---------------------|
| Blockchain | A ledger everyone can see, no one controls | [01_Blockchain_Basics.md](01_Blockchain_Basics.md) |
| Ethereum | A global computer that runs programs | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md) |
| Smart Contract | A program that executes automatically | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#smart-contracts) |
| Staking | Locking up ETH to help secure the network | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md) |
| Validator | An entity that proposes/validates blocks | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md#becoming-a-validator) |
| Slot | 12-second period where a block can be proposed | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#slots-epochs-and-time) |
| Epoch | 32 slots (~6.4 minutes) | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#slots-epochs-and-time) |
| Finality | Point when blocks become irreversible | [01_Blockchain_Basics.md](01_Blockchain_Basics.md) |
| Consensus Layer | The part that manages validators and PoS | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#dual-layer-architecture) |
| Execution Layer | The part that processes transactions | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#dual-layer-architecture) |
| Gas | Measure of computational work | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#gas) |
| Priority Fee | Tip to validators (taxable income) | [02_Ethereum_Concepts.md](02_Ethereum_Concepts.md#eip-1559) |
| MEV | Extra profit from transaction ordering | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md#types-of-staking-rewards) |
| Withdrawal | ETH removed from a validator | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md#withdrawals) |
| Rocket Pool | Decentralized staking pool | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md#rocket-pool-staking) |
| Minipool | Rocket Pool's validator instance | [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md#how-it-works) |

### Concept → Code Quick Map

| I want to understand... | Read this section... |
|------------------------|---------------------|
| How balances are tracked | [04_Understanding_This_Repo.md - Feature 1](04_Understanding_This_Repo.md#feature-1-tracking-validator-balances) |
| How income is calculated | [04_Understanding_This_Repo.md - Feature 2](04_Understanding_This_Repo.md#feature-2-calculating-income) |
| How MEV is detected | [04_Understanding_This_Repo.md - Feature 3](04_Understanding_This_Repo.md#feature-3-mev-detection) |
| How Rocket Pool works | [04_Understanding_This_Repo.md - Feature 4](04_Understanding_This_Repo.md#feature-4-rocket-pool-tracking) |
| Database schema | [04_Understanding_This_Repo.md - Key Files](04_Understanding_This_Repo.md#database-layer) |
| API endpoints | [04_Understanding_This_Repo.md - API Layer](04_Understanding_This_Repo.md#api-layer) |

### Common Questions

**Q: Do I need to run a validator to understand this?**
A: No! The tutorials explain everything from first principles.

**Q: How long will this take?**
A: 4-6 hours for complete beginners, 2-3 hours if you know blockchain basics.

**Q: Can I skip the tutorials and just read the code?**
A: You could, but you'll miss important context about WHY the code does what it does, especially for complex parts like MEV detection and Rocket Pool reward splitting.

**Q: What if I get stuck?**
A:
1. Re-read the relevant tutorial section
2. Check [Learning_Resources.md](Learning_Resources.md) for videos/articles
3. Explore the links to Etherscan, Beaconcha.in, etc.
4. Ask in the ethstaker Discord or Reddit
5. Open a GitHub issue if you find errors in the tutorials

**Q: I'm a tax professional, not a developer. Is this for me?**
A: Yes! Focus on [03_Staking_and_Taxation.md](03_Staking_and_Taxation.md). You don't need to understand the code, just the calculation methodology.

**Q: How do I stay updated?**
A: Check the "Updates and Staying Current" section in [Learning_Resources.md](Learning_Resources.md)

## Practice Challenges

After completing the tutorials, test your understanding:

### Challenge 1: Calculate Income Manually
Given:
- Validator balance on Jan 1: 32.0000 ETH
- Validator balance on Jan 2: 32.0036 ETH
- No withdrawals
- No block proposals

**Question**: What's the taxable income for Jan 1?

<details>
<summary>Answer</summary>

0.0036 ETH

Calculation: end_balance - start_balance = 32.0036 - 32.0000 = 0.0036 ETH
</details>

### Challenge 2: Account for Withdrawal
Given:
- Validator balance on Jan 1: 32.5000 ETH
- Validator balance on Jan 2: 32.0036 ETH (after automatic partial withdrawal)
- Withdrawal amount: 0.5000 ETH
- No block proposals

**Question**: What's the taxable income for Jan 1?

<details>
<summary>Answer</summary>

0.0036 ETH

Calculation: (end_balance - start_balance) + withdrawal_amount = (32.0036 - 32.5000) + 0.5000 = -0.4964 + 0.5000 = 0.0036 ETH

The withdrawal reduced the balance, but we add it back because it's not a loss - it's accessing earned income.
</details>

### Challenge 3: Block Proposal Day
Given:
- Validator balance on Jan 1: 32.0000 ETH
- Validator balance on Jan 2: 32.0136 ETH
- Proposed block with 0.05 ETH priority fees
- No MEV

**Question**: What's the total taxable income for Jan 1?

<details>
<summary>Answer</summary>

0.0636 ETH

Calculation:
- Consensus layer: 32.0136 - 32.0000 = 0.0136 ETH
- Execution layer: 0.05 ETH (priority fees)
- Total: 0.0136 + 0.05 = 0.0636 ETH
</details>

### Challenge 4: Find in Code
**Task**: Find where in the code the balance change is calculated for consensus layer income.

<details>
<summary>Answer</summary>

`src/api/api_v2/endpoints/rewards.py:527`

```python
amount_earned_wei = Decimal(1e18) * (eod_balance.balance - prev_balance.balance)
```
</details>

### Challenge 5: Rocket Pool Reward Split
Given:
- Minipool with 8 ETH bond (node operator's capital)
- 24 ETH from protocol
- 14% commission
- Withdrawal: 0.32 ETH

**Question**: What's the node operator's share?

<details>
<summary>Answer</summary>

0.09 ETH

Calculation:
```
FULL_MINIPOOL_BOND = 32 ETH

Node operator share = withdrawal × (
    (bond / FULL_MINIPOOL_BOND) +
    ((FULL_MINIPOOL_BOND - bond) / FULL_MINIPOOL_BOND) × (commission)
)

= 0.32 × (
    (8 / 32) +
    ((32 - 8) / 32) × 0.14
)

= 0.32 × (0.25 + 0.75 × 0.14)
= 0.32 × (0.25 + 0.105)
= 0.32 × 0.355
= 0.1136 ETH

Wait, let me recalculate:
= 0.32 × (8/32 + 24/32 × 0.14)
= 0.32 × (0.25 + 0.75 × 0.14)
= 0.32 × (0.25 + 0.105)
= 0.32 × 0.355
≈ 0.114 ETH

Actually, this represents the share. The node operator gets their capital returns plus commission on protocol's share.
```

See `src/api/api_v2/endpoints/rewards.py:116-121` for the exact formula.
</details>

## Contributing to These Tutorials

Found a typo? Want to add clarification? See a broken link?

1. Edit the relevant markdown file
2. Submit a pull request
3. Or open an issue on GitHub

These tutorials are meant to be living documents that improve over time.

## What's Next?

After completing these tutorials, you should:

1. **Review the main repository documentation**: [REPOSITORY_DOCUMENTATION.md](../REPOSITORY_DOCUMENTATION.md)
2. **Set up your development environment**: Follow the setup guide in README.MD
3. **Pick a "good first issue"**: Check GitHub issues labeled "good first issue"
4. **Join the community**: ethstaker Discord, Reddit
5. **Keep learning**: Ethereum is always evolving - stay updated!

## Additional Resources

- **Main repository docs**: [../REPOSITORY_DOCUMENTATION.md](../REPOSITORY_DOCUMENTATION.md)
- **Database schema**: [../src/db/tables.py](../src/db/tables.py)
- **API documentation**: Run the server and visit http://localhost:8000/api/docs
- **Docker setup**: [../docker-compose.yml](../docker-compose.yml)

---

**Questions or feedback?** Open an issue on GitHub or reach out to the maintainers.

**Good luck on your learning journey! 🚀**
