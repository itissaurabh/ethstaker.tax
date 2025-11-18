# Ethereum Concepts: Beyond Simple Transactions

## Table of Contents
1. [The Intuitive Understanding](#the-intuitive-understanding)
2. [Ethereum's Key Innovation: Smart Contracts](#ethereums-key-innovation-smart-contracts)
3. [The Ethereum Virtual Machine (EVM)](#the-ethereum-virtual-machine-evm)
4. [Accounts, Gas, and Transactions](#accounts-gas-and-transactions)
5. [The Dual-Layer Architecture](#the-dual-layer-architecture)
6. [Technical Deep Dive](#technical-deep-dive)
7. [Why This Matters for ethstaker.tax](#why-this-matters-for-ethstakertax)

---

## The Intuitive Understanding

### Bitcoin vs Ethereum: The Essential Difference

**Bitcoin**:
- Think of it as a **calculator**
- Does one thing very well: tracks who owns how many bitcoins
- Transactions are simple: "Send X bitcoins from A to B"

**Ethereum**:
- Think of it as a **computer**
- Can run programs (called "smart contracts")
- Transactions can be: "Run this program with these inputs"

**Analogy**:
- Bitcoin = a very secure spreadsheet
- Ethereum = a very secure spreadsheet that can also run Excel macros

### What is Ethereum?

**Simple definition**: Ethereum is a global computer that:
- Runs 24/7
- Can't be shut down
- Executes programs exactly as written
- Is run by thousands of independent computers

**Why this matters**: You can create applications that:
- Don't have a single owner (decentralized apps)
- Can't be censored or stopped
- Execute automatically without human intervention
- Are transparent (anyone can verify the code)

---

## Ethereum's Key Innovation: Smart Contracts

### What is a Smart Contract?

**Simple analogy**: A vending machine

**Regular contract**:
```
"If you give me $2, I promise to give you a soda"
Problem: Requires trust. What if I don't give you the soda?
```

**Smart contract (vending machine)**:
```
IF (money inserted == $2):
    dispense soda
    no refund possible
ELSE:
    return money
```

The machine enforces the contract automatically. No trust needed.

### Real Smart Contract Example: A Bet

Alice and Bob want to bet on tomorrow's weather:
- If it rains: Alice gets 2 ETH
- If it doesn't: Bob gets 2 ETH

**Traditional approach**:
1. Alice and Bob each give $100 to a trusted friend
2. Friend checks the weather tomorrow
3. Friend gives $200 to the winner
4. Problem: Have to trust the friend

**Smart contract approach**:
```solidity
contract WeatherBet {
    address alice;
    address bob;

    // Alice and Bob each deposit 1 ETH (totalling 2 ETH)
    function deposit() public payable { }

    // Oracle reports if it rained
    function resolve(bool didItRain) public {
        if (didItRain) {
            alice.transfer(2 ETH);
        } else {
            bob.transfer(2 ETH);
        }
    }
}
```

Benefits:
- No trusted third party needed (the code is the referee)
- Can't be cheated (code always executes the same way)
- Transparent (anyone can see the rules)

### Smart Contracts in the Real World

**DeFi (Decentralized Finance)**:
- **Lending**: Smart contracts that let you borrow/lend money
- **Trading**: Smart contracts that exchange tokens automatically
- **Staking**: Smart contracts that lock your ETH and give you rewards

**NFTs (Non-Fungible Tokens)**:
- Smart contracts that represent ownership of unique items
- Digital art, collectibles, domain names

**DAOs (Decentralized Autonomous Organizations)**:
- Smart contracts that govern groups/companies
- Members vote, smart contract executes decisions

**Rocket Pool** (relevant to this repo):
- Smart contracts that manage validator deposits
- Automatically distribute rewards between node operators and stakers
- This is why Rocket Pool reward calculation is complex!

---

## The Ethereum Virtual Machine (EVM)

### What is the EVM?

**Simple explanation**: The EVM is like the CPU of the Ethereum computer.

**More precisely**: The EVM is the program that:
1. Reads smart contract code (bytecode)
2. Executes it step-by-step
3. Updates Ethereum's state (balances, contract storage)

**Analogy**: Think of the EVM as:
- Smart contract code = recipe
- EVM = chef who follows the recipe exactly
- Ethereum state = ingredients and final dish

### How the EVM Works

**Every Ethereum node runs the EVM**:
```
Transaction arrives → EVM executes code → State changes
```

**Example**:
```
Transaction: "Call contract XYZ with function transfer(Bob, 10)"
EVM:
  1. Load contract XYZ's code
  2. Run the 'transfer' function
  3. Check: Does sender have 10 tokens?
  4. If yes: Update balances (subtract 10 from sender, add 10 to Bob)
  5. If no: Revert (undo everything)
```

**Key property**: Determinism
- Same input → Same output
- Every node running the same transaction gets the same result
- This is how consensus works!

### Smart Contract Languages

**Solidity** (most popular):
```solidity
contract SimpleStorage {
    uint256 public storedData;

    function set(uint256 x) public {
        storedData = x;
    }

    function get() public view returns (uint256) {
        return storedData;
    }
}
```

This compiles to **bytecode** that the EVM understands:
```
PUSH1 0x60 PUSH1 0x40 MSTORE CALLVALUE DUP1 ISZERO...
```

**You don't need to understand bytecode** - just know that:
- Humans write Solidity (readable)
- Compiler converts to bytecode (machine-readable)
- EVM executes bytecode

---

## Accounts, Gas, and Transactions

### Two Types of Accounts

**1. Externally Owned Account (EOA)** - Your wallet
- Has a private key (you control it)
- Can send transactions
- Can hold ETH
- Example: Your MetaMask wallet

**2. Contract Account** - A smart contract
- Has code (program logic)
- Can hold ETH
- Can't initiate transactions (must be called)
- Example: Uniswap contract, Rocket Pool minipool

**Key difference**:
- EOAs are controlled by people (via private keys)
- Contracts are controlled by their code

### Gas: The Fuel for Computation

**The problem**: Running code costs computational resources. How do we prevent spam and infinite loops?

**The solution**: Gas

**What is gas?**
- A measure of computational work
- Each operation costs gas:
  - Addition: 3 gas
  - Storage write: 20,000 gas
  - Transfer ETH: 21,000 gas

**How gas works**:
1. You set a **gas limit**: Maximum gas you're willing to use
2. You set a **gas price**: How much ETH per gas unit
3. Total fee = gas used × gas price

**Example**:
```
Transaction: Transfer 1 ETH
Gas needed: 21,000 gas
Gas price: 50 gwei (0.00000005 ETH)
Total fee: 21,000 × 50 gwei = 0.00105 ETH

Total cost: 1 ETH + 0.00105 ETH = 1.00105 ETH
```

### EIP-1559: Modern Gas Pricing

Since August 2021, Ethereum uses a new fee model:

**Base Fee** (burned):
- Adjusts based on network congestion
- More transactions → higher base fee
- Fewer transactions → lower base fee
- Gets destroyed (burned) - doesn't go to validators

**Priority Fee** (tip to validator):
- You set this as a "tip"
- Goes to the validator who includes your transaction
- Higher tip = faster inclusion

**Total fee** = (Base Fee + Priority Fee) × Gas Used

**Why this matters for ethstaker.tax**:
- Validators earn the priority fee (tips), not the base fee
- This is the `priority_fees_wei` in the database
- Base fee is burned and doesn't count as income

### Transaction Lifecycle

**Step 1: Create transaction**
```json
{
  "from": "0xAlice",
  "to": "0xBob",
  "value": "1000000000000000000",  // 1 ETH in wei
  "gas": "21000",
  "maxPriorityFeePerGas": "2000000000",  // 2 gwei
  "maxFeePerGas": "100000000000",  // 100 gwei
  "nonce": "5"
}
```

**Step 2: Sign transaction**
- Use your private key to create a digital signature
- Proves you authorized this transaction

**Step 3: Broadcast to network**
- Transaction goes to mempool (waiting area)
- Validators see it and may include it in next block

**Step 4: Validator includes in block**
- Validator chooses transactions (usually highest tips first)
- Executes all transactions
- Collects priority fees

**Step 5: Transaction finalized**
- After ~15 minutes, transaction is finalized
- Cannot be reversed

---

## The Dual-Layer Architecture

### Before "The Merge" (Pre-September 2022)

Ethereum was a single-layer system:
- Miners used Proof-of-Work (like Bitcoin)
- Mined blocks containing transactions
- This used huge amounts of electricity

### After "The Merge" (Post-September 2022)

Ethereum split into two layers:

**Execution Layer (EL)**:
- Processes transactions and smart contracts
- Maintains account balances
- Runs the EVM
- Formerly "Eth1" or "Mainnet"

**Consensus Layer (CL)**:
- Manages validators
- Decides which blocks are valid
- Achieves consensus through Proof-of-Stake
- Formerly "Beacon Chain" or "Eth2"

**How they work together**:
```
      Consensus Layer (CL)
      [Validators, PoS]
             ↓
      "This is the next block"
             ↓
      Execution Layer (EL)
      [Transactions, EVM]
             ↓
      "Block executed, state updated"
             ↓
      Consensus Layer confirms
```

### Why Split Into Two Layers?

**Separation of concerns**:
- CL focuses on security and consensus
- EL focuses on transaction execution
- Each can be optimized independently

**Client diversity**:
- Multiple CL clients: Lighthouse, Prysm, Teku, Nimbus, Lodestar
- Multiple EL clients: Geth, Nethermind, Besu, Erigon
- Can mix and match: Lighthouse + Geth, Prysm + Nethermind, etc.

**Benefits**:
- More resilient (bug in one client doesn't take down the network)
- More decentralized (no single implementation)
- Easier to upgrade (can upgrade each layer separately)

### How This Affects ethstaker.tax

The repository talks to **both layers**:

**Beacon Node** (Consensus Layer):
- `src/providers/beacon_node.py`
- Gets validator data, balances, attestations
- Queries like: "What was validator 123's balance at slot 1000?"

**Execution Node** (Execution Layer):
- `src/providers/execution_node.py`
- Gets block data, transactions, contract calls
- Queries like: "What was this address's balance at block 15,000,000?"

**Why both are needed**:
- Consensus layer rewards (attestations, proposals) → from beacon node
- Execution layer rewards (transaction fees, MEV) → from execution node
- Total income = CL rewards + EL rewards

---

## Technical Deep Dive

### Ethereum State

Ethereum maintains a **global state** that contains:

**Account state**:
```python
{
  "0xAlice": {
    "balance": "10000000000000000000",  # 10 ETH
    "nonce": 5,  # Number of transactions sent
    "code": "",  # Empty for EOAs
    "storage": {}  # Empty for EOAs
  },
  "0xContract": {
    "balance": "50000000000000000000",  # 50 ETH
    "nonce": 1,
    "code": "0x608060...",  # Smart contract bytecode
    "storage": {
      "slot0": "0x123...",  # Contract's storage
      "slot1": "0x456..."
    }
  }
}
```

**State transitions**:
```
Old State + Transaction = New State
```

This is what validators do: apply transactions to update the state.

### Merkle Patricia Trie

Ethereum stores state in a **Merkle Patricia Trie** (MPT):

**Why not a simple database?**
- Need cryptographic proof of state
- Need to sync state efficiently
- Need to prove "account X has balance Y" without showing all accounts

**How it works**:
```
State Root Hash (32 bytes)
     ↓
Represents the entire Ethereum state (millions of accounts)
     ↓
Can prove any account's balance using a "Merkle proof"
```

**Example proof**:
"Prove Alice has 10 ETH"
→ Provide ~10 hashes from state root to Alice's account
→ Anyone can verify without downloading the entire state

### Validator Rewards: The Full Picture

**Consensus Layer Rewards**:
1. **Attestation rewards**: For voting on blocks (~every 6.4 minutes)
2. **Block proposal rewards**: For proposing a block (random, ~every few days)
3. **Sync committee rewards**: For being on sync committee (periodic, 27 hours)

**Execution Layer Rewards**:
1. **Priority fees**: Transaction tips when you propose a block
2. **MEV**: Extra profit from transaction ordering

**Penalties** (negative rewards):
1. **Inactivity leak**: Penalty for being offline when chain isn't finalizing
2. **Attestation penalties**: Small penalty for missing attestations
3. **Slashing**: Large penalty for provable misbehavior (very rare)

### Slots, Epochs, and Time

**Slot**: 12-second period
- One block can be proposed per slot
- ~7,200 slots per day

**Epoch**: 32 slots (~6.4 minutes)
- Attestation committees shuffled each epoch
- Rewards calculated per epoch
- Checkpoint for finality

**Timeline**:
```
Slot 0    Slot 1    Slot 2    ...    Slot 31   Slot 32   Slot 33
|---------|---------|---------|  ...  |---------|---------|---------|
←────────────────  Epoch 0 ──────────────────→←────── Epoch 1 ──────→
```

**Finality**: After 2 epochs (~13 minutes), blocks are finalized
- Finalized blocks cannot be reverted
- This is why `is_slot_finalized()` is important in the code

### Validators and Balances

**Effective Balance** vs **Actual Balance**:

**Effective Balance**:
- Used for consensus (voting power)
- Rounded down to nearest 1 ETH
- Capped at 32 ETH (currently)
- Only updates at epoch boundaries

**Actual Balance**:
- Precise balance including all rewards
- Updates every epoch
- Can exceed 32 ETH

**Example**:
```
Actual Balance: 32.7 ETH → Effective Balance: 32 ETH (voting power)
Actual Balance: 31.9 ETH → Effective Balance: 31 ETH (voting power reduced)
```

**Why this matters**:
- ethstaker.tax tracks actual balance (for income calculation)
- Rewards accumulate in actual balance
- After withdrawals enabled (April 2023), excess above 32 ETH auto-withdraws

---

## Why This Matters for ethstaker.tax

### 1. Understanding Two Reward Sources

The code calculates rewards from both layers:

**Consensus Layer** (`src/api/api_v2/endpoints/rewards.py:549`):
```python
consensus_layer_rewards[validator_index].append(
    RewardForDate.construct(
        date=date,
        amount_wei=amount_earned_wei,  # Balance change
    )
)
```

**Execution Layer** (`src/api/api_v2/endpoints/rewards.py:571`):
```python
execution_layer_rewards[validator_index].append(
    RewardForDate.construct(
        date=date,
        amount_wei=rewards_sum  # Priority fees + MEV
    )
)
```

### 2. Understanding Gas and Priority Fees

When your validator proposes a block, you earn:

```python
# src/db/tables.py:26
priority_fees_wei = Column(Numeric(precision=27), nullable=True)
```

This is the sum of all priority fees (tips) from transactions in your block.

**Why not base fees?**
- Base fees are burned (destroyed)
- Only priority fees go to validators
- This is taxable income for validators

### 3. Understanding Smart Contracts (Rocket Pool)

Rocket Pool uses smart contracts to:
- Pool deposits from multiple users
- Create validators
- Distribute rewards

This is why Rocket Pool logic is complex:
```python
# src/api/api_v2/endpoints/rewards.py:99-104
return total_reward_wei * (
    # NO bond part
    (bond / _FULL_MINIPOOL_BOND)
    # User bond part - commission
    + ((_FULL_MINIPOOL_BOND - bond) / _FULL_MINIPOOL_BOND) * (fee / Decimal(1e18))
)
```

The smart contract code defines how rewards are split.

### 4. Understanding Contract Calls

The code queries smart contracts:
```python
# src/providers/execution_node.py:65
async def eth_call(self, params: list[dict], use_infura=True) -> Any:
```

**Example**: Get Rocket Pool minipool fee
```python
# src/providers/rocket_pool.py:59
result = await self.execution_node.eth_call(
    params=[{
        "to": minipool_address,
        "data": "0xe7150134",  # getNodeFee() function
    }]
)
```

This calls the smart contract's `getNodeFee()` function.

### 5. Understanding Historical State Queries

To calculate past income, we need past balances:

```python
# Query balance at historical block
balance = await execution_node.get_balance(
    address=fee_distributor,
    block_number=block_number-1
)
```

**This requires an archive node** - a node that stores all historical state.

Regular nodes only keep recent state, so can't answer "What was this address's balance 6 months ago?"

### 6. Understanding Finality

The code only processes finalized data:

```python
# src/indexer/balances.py:117
if not await beacon_node.is_slot_finalized(slot):
    logger.info(f"Waiting for slot {slot} to be finalized")
    continue
```

**Why?**
- Unfinalized blocks can be reverted (reorganized)
- Tax calculations need permanent data
- Finalized = guaranteed to never change

---

## Common Misconceptions

### ❌ "Smart contracts are stored on-chain"
**✓ Truth**: Smart contract **bytecode** is stored on-chain. The Solidity source code is not (unless verified on Etherscan).

### ❌ "Gas fees go to miners/validators"
**✓ Truth**: Only the **priority fee** goes to validators. The **base fee** is burned (destroyed forever).

### ❌ "Ethereum is slow"
**✓ Truth**: Ethereum prioritizes security and decentralization over speed. Layer 2 solutions (Arbitrum, Optimism) provide speed.

### ❌ "You need ETH to receive ETH"
**✓ Truth**: You need ETH to **send** transactions, not to receive. Your address can receive ETH even if it has 0 balance.

### ❌ "Smart contracts can be changed"
**✓ Truth**: Once deployed, smart contract code is **immutable**. However, contracts can be designed with upgrade mechanisms.

### ❌ "The Merge made Ethereum faster"
**✓ Truth**: The Merge made Ethereum more energy-efficient and secure. Speed stayed roughly the same (12-second blocks).

---

## Practical Exercises

### Exercise 1: Explore Ethereum

1. Go to https://etherscan.io/
2. Look at a recent block
3. Identify:
   - Block number and timestamp
   - Number of transactions
   - Base fee (burned)
   - Priority fee total (paid to validator)
4. Click on a transaction:
   - What was transferred?
   - How much gas was used?
   - What was the fee?

### Exercise 2: Smart Contract Interaction

1. Go to https://etherscan.io/address/0xd4e96ef8eee8678dbff4d535e033ed1a4f7605b7
   (This is Rocket Pool's Smoothing Pool contract)
2. Click "Contract" tab
3. See the source code (Solidity)
4. Click "Read Contract"
5. Try calling view functions (no cost, just reading data)

### Exercise 3: Track a Validator

1. Go to https://beaconcha.in/
2. Enter a validator index (try: 1)
3. See:
   - Balance over time
   - Attestations
   - Block proposals
   - Income chart

### Exercise 4: Gas Tracker

1. Go to https://etherscan.io/gastracker
2. See current gas prices
3. Notice:
   - Base fee varies
   - Priority fees recommended for different speeds
4. Check during different times of day - notice patterns

---

## Summary

**What you should now understand**:

1. **Ethereum is a global computer** that runs smart contracts
2. **Smart contracts** are programs that execute automatically without intermediaries
3. **The EVM** executes smart contract code deterministically
4. **Gas** prevents spam and pays for computation
5. **Two account types**: EOAs (users) and Contracts (programs)
6. **Dual-layer architecture**: Consensus Layer (PoS) + Execution Layer (EVM)
7. **Validator rewards** come from both layers
8. **State** is the current balances and data, stored in a Merkle Trie
9. **Finality** ensures transactions can't be reversed

**How this connects to ethstaker.tax**:
- Queries both CL (balances, attestations) and EL (fees, contracts)
- Calculates income from both reward sources
- Handles smart contracts (Rocket Pool)
- Requires archive data for historical queries
- Only processes finalized data for accuracy

**Next steps**:
- Read `03_Staking_and_Taxation.md` to understand how staking generates taxable income
- Try the practical exercises above
- Watch recommended videos in `Learning_Resources.md`

---

## Glossary

- **Base Fee**: Portion of gas fee that gets burned (destroyed)
- **Beacon Chain**: The consensus layer (formerly called Eth2)
- **Bytecode**: Machine-readable smart contract code
- **Consensus Layer (CL)**: Layer managing validators and consensus
- **Contract Account**: An account containing smart contract code
- **EIP**: Ethereum Improvement Proposal
- **EVM**: Ethereum Virtual Machine - executes smart contracts
- **Execution Layer (EL)**: Layer processing transactions and smart contracts
- **Externally Owned Account (EOA)**: A user-controlled account
- **Finality**: Point when blocks become irreversible (~15 minutes)
- **Gas**: Measure of computational work
- **Gwei**: 1 billionth of an ETH (common unit for gas prices)
- **Mainnet**: The main Ethereum network
- **Mempool**: Waiting area for unconfirmed transactions
- **Priority Fee**: Tip to validators (taxable income)
- **Smart Contract**: Self-executing program on Ethereum
- **Solidity**: Popular smart contract programming language
- **State**: Current balances and data in Ethereum
- **Wei**: Smallest unit of ETH (1 ETH = 10^18 wei)

---

**Continue to**: `03_Staking_and_Taxation.md` to learn about Ethereum staking and tax calculations.
