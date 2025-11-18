# Understanding the ethstaker.tax Repository

## Table of Contents
1. [Connecting the Concepts to Code](#connecting-the-concepts-to-code)
2. [Repository Architecture](#repository-architecture)
3. [Code Walkthrough by Feature](#code-walkthrough-by-feature)
4. [Key Files and Their Purpose](#key-files-and-their-purpose)
5. [Common Maintenance Tasks](#common-maintenance-tasks)
6. [Debugging Guide](#debugging-guide)
7. [Extension Points](#extension-points)

---

## Connecting the Concepts to Code

Now that you understand blockchain, Ethereum, and staking, let's see how these concepts are implemented in this repository.

### Concept → Code Mapping

| Concept | Where It's Implemented |
|---------|----------------------|
| **Blockchain basics** | |
| - Slots and timestamps | `src/providers/beacon_node.py:71-89` |
| - Block hashes | Used in queries but not stored |
| - Finality | `src/indexer/balances.py:117` |
| **Ethereum concepts** | |
| - Consensus Layer | `src/providers/beacon_node.py` (entire file) |
| - Execution Layer | `src/providers/execution_node.py` (entire file) |
| - Gas and fees | `src/db/tables.py:26` (priority_fees_wei) |
| - Smart contracts | `src/providers/rocket_pool.py` (eth_call usage) |
| - State queries | `execution_node.py:65` (eth_call) |
| **Staking concepts** | |
| - Validator balances | `src/indexer/balances.py` |
| - Block proposals | `src/indexer/block_rewards/main.py` |
| - Withdrawals | `src/indexer/withdrawals.py` |
| - Activation slots | `beacon_node.py:activation_slots_for_validators()` |
| **Taxation concepts** | |
| - Daily income calculation | `src/api/api_v2/endpoints/rewards.py:549` |
| - Balance tracking | `src/db/tables.py:9-14` (Balance table) |
| - Price conversion | `src/indexer/prices.py` |
| - MEV detection | `src/indexer/block_rewards/block_rewards_mev_simple.py` |
| **Rocket Pool** | |
| - Minipool tracking | `src/indexer/rocket_pool/main.py` |
| - Reward splitting | `src/api/api_v2/endpoints/rewards.py:99-121` |
| - Bond reductions | `src/db/tables.py:43-52` |

---

## Repository Architecture

### The Big Picture

```
User Request (Web/API)
        ↓
    API Server (FastAPI)
        ↓
    Database Queries (DbProvider)
        ↓
    PostgreSQL Database
        ↑
    (populated by)
        ↑
    Indexers (background processes)
        ↑
    (query data from)
        ↑
    Providers (BeaconNode, ExecutionNode, etc.)
        ↑
    (connect to)
        ↑
    External Services (CL client, EL client, APIs)
```

### Process Flow: End-to-End

**1. Data Collection** (Indexers):
```
Indexer starts
  → Queries beacon/execution node
  → Processes raw data
  → Writes to PostgreSQL
  → Repeats (infinite loop with sleep)
```

**2. Data Storage** (Database):
```
PostgreSQL stores:
  - Validator balances (daily + activation)
  - Block rewards (priority fees, MEV)
  - Withdrawals
  - Rocket Pool data
  - Price data
```

**3. Data Retrieval** (API):
```
User makes API request
  → FastAPI endpoint receives request
  → DbProvider queries database
  → Business logic calculates income
  → Response returned to user
```

---

## Code Walkthrough by Feature

### Feature 1: Tracking Validator Balances

**Purpose**: Record balance at end-of-day and activation for all validators

**Entry point**: `src/indexer/balances.py`

**Key function**: `index_balances()`

**Step-by-step**:

1. **Calculate which slots to index** (`balances.py:40-88`):
```python
# Activation slots for all validators
validator_index_to_activation_slot = await beacon_node.activation_slots_for_validators(...)

# End-of-day slots for each day
eod_slots = set()
for timezone in TIMEZONES_TO_INDEX:
    # Calculate slot for 23:59:59 each day
    slots.extend(beacon_node.slot_for_datetime(dt) for dt in datetimes)
```

**Why these specific slots?**
- Activation slot: Starting balance for income calculation
- End-of-day: Daily balance for income calculation
- UTC timezone: Tax days aligned with UTC

2. **Remove already-indexed slots** (`balances.py:91-101`):
```python
ALREADY_INDEXED_SLOTS = [s for s, in session.query(Balance.slot).distinct().all()]
slots_to_index = sorted(activation_slots.union(eod_slots))
# Remove already indexed
for s in ALREADY_INDEXED_SLOTS:
    if s in activation_slots:
        activation_slots.remove(s)
```

**Why?** Don't re-index data we already have. Saves time and API calls.

3. **Index each slot** (`balances.py:112-156`):
```python
for slot in slots_to_index:
    # Wait for finality
    if not await beacon_node.is_slot_finalized(slot):
        continue

    # Get balances
    if slot in eod_slots:
        balances_for_slot = await beacon_node.balances_for_slot(slot)
    else:  # Activation slot
        balances_for_slot = await beacon_node.balances_for_slot(
            slot=slot,
            validator_indexes=activation_slot_to_validators[slot]
        )

    # Insert into database
    session.execute(text("INSERT INTO balance(validator_index, slot, balance) ..."))
```

**Key concepts used**:
- **Finality** (blockchain): Only process finalized slots
- **State queries** (Ethereum): Get historical balances at specific slots
- **Database indexing**: Store for later retrieval

### Feature 2: Calculating Income

**Purpose**: Calculate total income for validators over a date range

**Entry point**: `src/api/api_v2/endpoints/rewards.py`

**Key function**: `rewards()` (full endpoint) at line 413

**Step-by-step**:

1. **Preprocess request** (`rewards.py:420`):
```python
validator_indexes, start_datetime, end_datetime, min_slot, max_slot, expected_fee_recipient_addresses = await _preprocess_request_input_data(rewards_request)
```

Converts user's date range to slot range.

2. **Get initial balances** (`rewards.py:436-460`):
```python
for validator_index in validator_indexes:
    act_slot = activation_slots[validator_index]

    if act_slot > first_slot_in_requested_period:
        # Validator activated during period - use activation balance
        initial_balance = db_provider.balances(slots=[act_slot], ...)
    else:
        # Use balance at start of period
        initial_balance = db_provider.balances(slots=[initial_balance_slot], ...)
```

**Why?** Starting point for calculating "balance change = income".

3. **Get end-of-day balances** (`rewards.py:463-475`):
```python
eod_slots = [
    beacon_node.slot_for_datetime(
        dt=datetime.datetime.combine(
            (rewards_request.start_date + datetime.timedelta(days=day_idx)),
            time=datetime.time(hour=23, minute=59, second=59),
            tzinfo=pytz.UTC,
        )
    )
    for day_idx in range(range_day_count)
]
eod_balances = db_provider.balances(slots=eod_slots, validator_indexes=validator_indexes)
```

**Why?** End point for calculating "balance change = income" for each day.

4. **Get withdrawals and block rewards** (`rewards.py:478-491`):
```python
all_withdrawals = sorted(db_provider.withdrawals(
    min_slot=min_slot,
    max_slot=max_slot,
    validator_indexes=validator_indexes
), key=lambda x: x.slot)

all_block_rewards = db_provider.block_rewards(
    min_slot=min_slot,
    max_slot=max_slot,
    proposer_indexes=validator_indexes
)
```

**Why?** Need to account for withdrawals (reduce balance but aren't losses) and execution layer income.

5. **Calculate consensus layer income** (`rewards.py:522-562`):
```python
for eod_balance in [eodb for eodb in eod_balances if eodb.validator_index == validator_index]:
    # Balance change
    amount_earned_wei = Decimal(1e18) * (eod_balance.balance - prev_balance.balance)

    # Add back withdrawals (they reduced balance but aren't losses)
    amount_withdrawn_this_day_wei = 0
    for w in validator_withdrawals:
        if eod_balance.slot >= w.slot > prev_balance.slot:
            if w.amount_gwei > 8 * Decimal(1e9):
                # Full withdrawal - don't count the principal (32 ETH)
                amount_withdrawn_this_day_wei += (w.amount_gwei % (32 * Decimal(1e9))) * Decimal(1e9)
            else:
                # Partial withdrawal - count it all
                amount_withdrawn_this_day_wei += w.amount_gwei * Decimal(1e9)

    amount_earned_wei += amount_withdrawn_this_day_wei

    consensus_layer_rewards[validator_index].append(
        RewardForDate.construct(date=date, amount_wei=amount_earned_wei)
    )
```

**Key taxation concept**: Withdrawals reduce balance but aren't losses - must add them back.

6. **Add execution layer income** (`rewards.py:565-576`):
```python
exec_layer_rewards_for_date = defaultdict(int)
for br in [br for br in all_block_rewards if br.proposer_index == validator_index]:
    exec_layer_rewards_for_date[
        beacon_node.datetime_for_slot(br.slot, pytz.UTC).date()
    ] += br.mev_reward_value_wei if br.mev else br.priority_fees_wei
```

**Key concept**: Priority fees + MEV = execution layer income.

### Feature 3: MEV Detection

**Purpose**: Determine if a block earned MEV and how much

**Entry point**: `src/indexer/block_rewards/block_rewards_mev_simple.py`

**Key function**: `get_block_reward_value()`

**The challenge**: MEV isn't explicitly in the protocol - must detect it

**Method 1: Check MEV relay** (lines ~100-150):
```python
try:
    delivered_payload = await mev_relay.get_delivered_payload(slot=slot_proposer_data.slot)
    if delivered_payload:
        # Block used MEV-Boost
        return BlockRewardValue(
            block_priority_tx_fees=...,
            contains_mev=True,
            mev_recipient=delivered_payload['proposer_fee_recipient'],
            mev_recipient_balance_change=delivered_payload['value']
        )
except:
    # Relay might be down or block not in relay
    pass
```

**Concept**: MEV-Boost relays keep records of MEV blocks.

**Method 2: Analyze balance changes** (if relay doesn't have data):
```python
# Get fee recipient's balance change
balance_change = (
    await execution_node.get_balance(fee_recipient, block_number) -
    await execution_node.get_balance(fee_recipient, block_number - 1)
)

# Calculate expected priority fees
expected_fees = sum(tx.priority_fee for tx in block.transactions)

# If balance change > expected, likely MEV
if balance_change > expected_fees:
    mev_value = balance_change - expected_fees
```

**Concept**: MEV shows up as "extra" balance beyond transaction fees.

### Feature 4: Rocket Pool Tracking

**Purpose**: Track Rocket Pool minipools, bonds, and rewards

**Entry point**: `src/indexer/rocket_pool/main.py`

**Key function**: `run()`

**Step-by-step**:

1. **Index nodes** (`main.py:53-66`):
```python
known_node_addresses = [a for a, in session.query(RocketPoolNode.node_address).all()]
rp_nodes = await rocket_pool_data.get_nodes(known_node_addresses=known_node_addresses, ...)

for node_address, fee_distributor in rp_nodes:
    if node_address in known_node_addresses:
        continue
    session.add(RocketPoolNode(
        node_address=node_address,
        fee_distributor=fee_distributor,
    ))
```

**Concept**: Rocket Pool nodes are registered in smart contracts. Query the contracts.

2. **Index minipools** (`main.py:74-91`):
```python
for node_address, minipool_list in (await rocket_pool_data.get_minipools(...)).items():
    for minipool_address, pubkey, initial_bond_value, initial_fee_value in minipool_list:
        session.add(RocketPoolMinipool(
            minipool_address=minipool_address,
            validator_pubkey=pubkey,
            initial_bond_value=initial_bond_value,
            initial_fee_value=initial_fee_value,
            node_address=node_address,
        ))
```

**Concept**: Minipools are created via events on the Rocket Pool contracts. Scan for `MinipoolCreated` events.

3. **Index bond reductions** (`main.py:95-112`):
```python
bond_reductions = await rocket_pool_data.get_bond_reductions(
    from_block_number=LAST_BLOCK_NUMBER_INDEXED,
    to_block_number=current_exec_block_number
)
for minipool_address, br_event_datetime, new_bond_amount, new_fee in bond_reductions:
    session.merge(RocketPoolBondReduction(
        minipool_address=minipool_address,
        timestamp=br_event_datetime,
        new_bond_amount=new_bond_amount,
        new_fee=new_fee
    ))
```

**Concept**: Bond reductions are events emitted by minipool contracts.

4. **Index reward snapshots** (`main.py:115-134`):
```python
new_reward_trees = await rocket_pool_data.get_reward_snapshots(start_at_period=last_indexed_reward_period+1)

for reward_period_index, node_rewards, period_end_time in new_reward_trees:
    session.add(RocketPoolRewardPeriod(
        reward_period_index=reward_period_index,
        reward_period_end_time=period_end_time,
        rewards=[
            RocketPoolReward(
                node_address=node_address,
                reward_collateral_rpl=node_rewards_data["collateralRpl"],
                reward_smoothing_pool_wei=node_rewards_data["smoothingPoolEth"],
            )
            for node_address, node_rewards_data in node_rewards.items()
        ]
    ))
```

**Concept**: Rocket Pool publishes reward merkle trees to IPFS every 28 days. Fetch and parse them.

---

## Key Files and Their Purpose

### Database Layer

**`src/db/tables.py`**:
- Defines database schema using SQLAlchemy ORM
- Tables: Balance, BlockReward, Withdrawal, Validator, Rocket Pool tables, Price
- **Modify this when**: Adding new data to track (e.g., new validator types, new withdrawal types)

**`src/db/db_helpers.py`**:
- Database utilities
- `session_scope()`: Context manager for database transactions
- **Use this**: When you need to interact with the database

### Provider Layer

**`src/providers/beacon_node.py`**:
- Interfaces with Consensus Layer (beacon chain)
- Key methods:
  - `balances_for_slot()`: Get validator balances
  - `activation_slots_for_validators()`: Get when validators activated
  - `withdrawals_for_slot()`: Get withdrawals
  - `slot_for_datetime()` / `datetime_for_slot()`: Convert between time and slots
- **Modify this when**: Beacon API changes, new consensus features

**`src/providers/execution_node.py`**:
- Interfaces with Execution Layer (EVM chain)
- Key methods:
  - `get_block()`: Get block data
  - `get_balance()`: Get ETH balance at specific block
  - `eth_call()`: Call smart contract functions
- **Modify this when**: New RPC methods needed, execution layer changes

**`src/providers/rocket_pool.py`**:
- Interfaces with Rocket Pool smart contracts
- Parses events, calls contract functions
- **Modify this when**: Rocket Pool upgrades contracts, new minipool manager versions

**`src/providers/db_provider.py`**:
- Centralized database query interface
- Abstracts database queries from business logic
- **Modify this when**: Need new database queries

### Indexer Layer

**`src/indexer/balances.py`**:
- Indexes validator balances daily
- Runs continuously in Docker container
- **Modify this when**: Need different indexing frequency, additional balance data

**`src/indexer/block_rewards/main.py`**:
- Indexes execution layer rewards (fees + MEV)
- Runs continuously, processes all slots
- **Modify this when**: New reward types, different indexing strategy

**`src/indexer/withdrawals.py`**:
- Indexes withdrawals (post-Shanghai)
- Runs continuously
- **Modify this when**: Need additional withdrawal metadata

**`src/indexer/validators.py`**:
- Indexes validator metadata (index ↔ pubkey mapping)
- Runs hourly
- **Modify this when**: Need additional validator metadata

**`src/indexer/rocket_pool/main.py`**:
- Indexes Rocket Pool specific data
- **Modify this when**: Rocket Pool protocol changes

**`src/indexer/prices.py`**:
- Fetches ETH prices from CoinGecko
- **Modify this when**: Need different price sources or currencies

### API Layer

**`src/api/app.py`**:
- FastAPI application entry point
- Sets up middleware, plugins, routes
- **Modify this when**: Adding new middleware, changing CORS, rate limiting

**`src/api/api_v2/endpoints/rewards.py`**:
- Core reward calculation logic
- Endpoints: `/rewards/full`, `/rewards/rocket_pool`
- **Modify this when**: Changing income calculation logic, adding features

**`src/api/api_v2/endpoints/prices.py`**:
- Price data endpoint
- **Modify this when**: Changing price data access

### Configuration

**`docker-compose.yml`**:
- Defines all services
- Environment variables
- **Modify this when**: Adding services, changing configurations

**`requirements.in`**:
- Python dependencies (human-edited)
- **Modify this when**: Adding/removing dependencies
- Then run: `make compile-dependencies`

**`alembic.ini`** + **`alembic/versions/*.py`**:
- Database migrations
- **Modify this when**: Changing database schema (use `make migration-generate`)

---

## Common Maintenance Tasks

### Task 1: Adding a New Rocket Pool Minipool Manager Version

**Scenario**: Rocket Pool deploys a new minipool manager contract

**Files to modify**:

1. **`src/providers/rocket_pool.py`**:
```python
_MINIPOOL_MANAGER_ADDRESSES = [
    # ... existing
    {
        # v6 deployed on YYYY-MM-DD
        "address": "0xNEWADDRESS",
    },
]
```

2. **Test**:
```bash
# Restart the Rocket Pool indexer
docker-compose restart indexer_rocket_pool

# Watch logs
docker-compose logs -f indexer_rocket_pool

# Should see new minipools being indexed
```

### Task 2: Adding Support for a New Withdrawal Type

**Scenario**: EIP-7251 adds consolidation withdrawals

**Files to modify**:

1. **`src/db/tables.py`**:
```python
class Withdrawal(Base):
    # ... existing fields
    withdrawal_type = Column(String(20), nullable=True)  # 'partial', 'full', 'consolidation'
```

2. **Generate migration**:
```bash
make migration-generate MIGRATION_NAME="add withdrawal type column"
# Review the generated migration in alembic/versions/
make migrate
```

3. **`src/indexer/withdrawals.py`**:
```python
# Add logic to classify withdrawal type
withdrawal_type = classify_withdrawal(withdrawal_amount, validator_status)

session.execute(text(
    "INSERT INTO withdrawal(slot, validator_index, amount_gwei, withdrawal_address_id, withdrawal_type)"
    " VALUES(:slot, :validator_index, :amount_gwei, :withdrawal_address_id, :withdrawal_type)"
), [...])
```

4. **`src/api/api_v2/endpoints/rewards.py`**:
```python
# Update income calculation to handle consolidation withdrawals
if w.withdrawal_type == 'consolidation':
    # Don't count as income
    continue
```

### Task 3: Changing the Balance Indexing Frequency

**Scenario**: Want to index balances twice daily instead of daily

**Files to modify**:

1. **`src/indexer/balances.py`**:
```python
# Change from end-of-day to twice daily
current_dt = start_dt.replace(hour=12, minute=0, second=0)
datetimes = []
while current_dt <= end_dt:
    # Noon
    datetimes.append(current_dt)
    # Midnight
    datetimes.append(current_dt.replace(hour=23, minute=59, second=59))
    current_dt += datetime.timedelta(days=1)
```

2. **`src/api/api_v2/endpoints/rewards.py`**:
```python
# Update EOD slot calculation to match
# Now need to aggregate 12:00 and 23:59 balances per day
```

### Task 4: Adding a New Currency

**Scenario**: Want to support EUR prices

**Files to modify**:

1. **`src/indexer/prices.py`**:
```python
# Add EUR to the currencies list
CURRENCIES = ['usd', 'eur']

# CoinGecko API already supports multiple currencies
```

2. **`src/api/api_v2/endpoints/prices.py`**:
```python
# No changes needed - already filters by currency parameter
```

3. **Update documentation for users**

---

## Debugging Guide

### Problem: Indexer is Not Progressing

**Symptoms**: Prometheus metric `slots_with_missing_balances` not decreasing

**Debugging steps**:

1. **Check if indexer is running**:
```bash
docker-compose ps indexer_balances
```

2. **Check logs**:
```bash
docker-compose logs -f indexer_balances

# Look for:
# - "Waiting for slot X to be finalized" (normal, just wait)
# - Exceptions (errors to fix)
# - "Indexing slot X" (progress indicators)
```

3. **Check if beacon node is synced**:
```bash
# Exec into beacon node container
docker-compose exec beacon_node lighthouse bn --network mainnet beacon-node-status

# Should show "Synced"
```

4. **Check database connection**:
```bash
# Try connecting to DB
docker-compose exec db psql -U <username> -d <dbname>

# Check if balance table exists
\dt

# Check row count
SELECT COUNT(*) FROM balance;
```

5. **Common causes**:
- Beacon node not synced yet
- Database full (out of disk space)
- Network issues (can't reach beacon node)
- Bug in indexing logic

### Problem: API Returns 500 Error

**Symptoms**: `/api/v2/rewards/full` returns HTTP 500

**Debugging steps**:

1. **Check API logs**:
```bash
docker-compose logs -f api

# Look for stack traces
```

2. **Check error message in response**:
```json
{
  "detail": "Execution layer rewards not available - missing data for proposer 12345, slot 6789012"
}
```

This tells you exactly what's wrong!

3. **Check if data is in database**:
```sql
-- Check if block reward exists
SELECT * FROM block_reward WHERE slot = 6789012;

-- Check if it was processed correctly
SELECT reward_processed_ok FROM block_reward WHERE slot = 6789012;
```

4. **Common causes**:
- Block reward indexer failed to process a slot (check `reward_processed_ok = false`)
- Execution node was down when indexing
- Database missing data

**Fix**: Re-index the problematic slot:
```bash
# Set environment variable to force re-indexing
docker-compose exec indexer_block_rewards \
  bash -c "export INDEX_ALL=true && python ./src/indexer/block_rewards/main.py"
```

### Problem: Incorrect Income Calculation

**Symptoms**: User reports income doesn't match expected

**Debugging approach**:

1. **Verify input data**:
```sql
-- Check balances
SELECT * FROM balance
WHERE validator_index = 12345
  AND slot BETWEEN 6000000 AND 6100000
ORDER BY slot;

-- Check withdrawals
SELECT * FROM withdrawal
WHERE validator_index = 12345;

-- Check block rewards
SELECT * FROM block_reward
WHERE proposer_index = 12345;
```

2. **Manual calculation**:
```python
# In a Python shell
start_balance = 32.0
end_balance = 32.001
withdrawal = 0.1

consensus_income = (end_balance - start_balance) + withdrawal
# Should be: (32.001 - 32.0) + 0.1 = 0.101 ETH
```

3. **Check edge cases**:
- Validator activated during the period?
- Full withdrawal (>8 ETH)?
- Multiple block proposals same day?
- Rocket Pool minipool?

4. **Add debug logging**:
```python
# In rewards.py
logger.debug(f"Processing validator {validator_index}")
logger.debug(f"Start balance: {prev_balance.balance}")
logger.debug(f"End balance: {eod_balance.balance}")
logger.debug(f"Withdrawals: {amount_withdrawn_this_day_wei}")
logger.debug(f"Calculated income: {amount_earned_wei}")
```

### Problem: MEV Not Detected

**Symptoms**: Block rewards show `mev = false` but user knows they used MEV-Boost

**Debugging**:

1. **Check MEV relay response**:
```python
# In a Python shell
from src.providers.mev_relay import MevRelay
import asyncio

mev_relay = MevRelay()
result = await mev_relay.get_delivered_payload(slot=6789012)
print(result)  # Should show payload if MEV was used
```

2. **If relay doesn't have data**:
- Check if block used a relay that's in our list
- Check `block_extra_data` for relay signature
- Use balance change analysis as fallback

3. **Check fee recipient**:
```sql
SELECT fee_recipient, mev, mev_reward_recipient
FROM block_reward
WHERE slot = 6789012;
```

If fee recipient is a smart contract (Rocket Pool, Lido), special handling is needed.

---

## Extension Points

### Adding a New Liquid Staking Protocol

**Similar to Rocket Pool support, but for e.g., Lido**

**Steps**:

1. **Understand the protocol**:
- How are validators created?
- How are rewards distributed?
- What smart contracts are involved?

2. **Add database tables**:
```python
# src/db/tables.py
class LidoValidator(Base):
    __tablename__ = "lido_validator"
    validator_index = Column(Integer, primary_key=True)
    # ... protocol-specific fields
```

3. **Create provider**:
```python
# src/providers/lido.py
class LidoDataProvider:
    async def get_validators(self):
        # Query Lido contracts
        pass
```

4. **Create indexer**:
```python
# src/indexer/lido/main.py
async def run():
    # Index Lido-specific data
    pass
```

5. **Add API endpoint**:
```python
# src/api/api_v2/endpoints/rewards.py
@router.post("/rewards/lido")
async def lido_rewards(...):
    # Calculate Lido operator rewards
    pass
```

6. **Add to docker-compose.yml**:
```yaml
indexer_lido:
  image: eth2-tax:latest
  command: ["python", "./src/indexer/lido/main.py"]
```

### Adding Real-time Updates

**Current**: Data updates every few minutes (indexer loops)

**Enhancement**: WebSocket API for real-time balance updates

**Steps**:

1. **Add WebSocket support to FastAPI**:
```python
# src/api/app.py
from fastapi import WebSocket

@app.websocket("/ws/balance/{validator_index}")
async def balance_websocket(websocket: WebSocket, validator_index: int):
    await websocket.accept()
    while True:
        balance = get_latest_balance(validator_index)
        await websocket.send_json({"balance": balance})
        await asyncio.sleep(12)  # Every slot
```

2. **Update indexers to publish updates**:
```python
# src/indexer/balances.py
import redis

redis_client = redis.Redis()

# After inserting balance
redis_client.publish(f"balance_update:{validator_index}", str(balance))
```

3. **Frontend subscribes to WebSocket**

### Adding Support for Multiple Chains

**Current**: Ethereum mainnet only

**Enhancement**: Support Gnosis Chain, other PoS chains

**Steps**:

1. **Add chain ID to all tables**:
```python
# src/db/tables.py
class Balance(Base):
    chain_id = Column(Integer, primary_key=True, default=1)  # 1 = Ethereum
    slot = Column(Integer, primary_key=True)
    validator_index = Column(Integer, primary_key=True)
```

2. **Parameterize providers**:
```python
# src/providers/beacon_node.py
class BeaconNode:
    def __init__(self, chain_id: int = 1):
        self.chain_id = chain_id
        self.BASE_URL = self._get_base_url_for_chain(chain_id)
```

3. **Update API to accept chain ID**:
```python
@router.post("/rewards/full")
async def rewards(
    chain_id: int = 1,
    rewards_request: RewardsRequest,
    ...
):
    # Filter all queries by chain_id
    pass
```

4. **Deploy separate indexers per chain**

---

## Summary

You now understand:

1. **How concepts map to code**: Each blockchain/Ethereum/staking concept has a specific implementation
2. **Repository architecture**: Indexers → Database → API flow
3. **Key files**: Where to find balance indexing, reward calculation, MEV detection, Rocket Pool logic
4. **Maintenance tasks**: How to add new features, modify existing logic
5. **Debugging**: How to diagnose and fix common issues
6. **Extension points**: How to add new protocols, features

**Next steps for maintenance**:
1. Review the actual code files mentioned above
2. Try running the stack locally
3. Make a small change and test it
4. Review open issues on GitHub
5. Read through the test files to understand edge cases

**You're now equipped to maintain and extend this repository!**
