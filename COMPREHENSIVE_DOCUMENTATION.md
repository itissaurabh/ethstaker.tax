# ETHstaker.tax - Comprehensive Developer Documentation

## Table of Contents
1. [What This Repository Does](#what-this-repository-does)
2. [Architecture Overview](#architecture-overview)
3. [Libraries and Dependencies](#libraries-and-dependencies)
4. [Function Call Flow](#function-call-flow)
5. [Detailed Function Descriptions](#detailed-function-descriptions)
6. [Taxation Logic Overview](#taxation-logic-overview)
7. [GitHub Issues Analysis](#github-issues-analysis)
8. [Adding Network Upgrade Support](#adding-network-upgrade-support)
9. [CL+EL Stack Requirements](#clel-stack-requirements)
10. [API Code Deep Dive](#api-code-deep-dive)

---

## What This Repository Does

**ETHstaker.tax** is a comprehensive Ethereum staking rewards and taxation tracking system. It provides:

### Core Functionality
- **Validator Rewards Tracking**: Indexes and tracks consensus layer (CL) rewards for Ethereum validators
- **Execution Layer Rewards**: Tracks priority fees and MEV rewards from block proposals
- **Rocket Pool Support**: Specialized tracking for Rocket Pool node operators
- **Price Indexing**: Historical ETH price data for tax calculations
- **Tax Reporting**: Generates tax reports with income calculations in various fiat currencies
- **Web Interface**: Vue3 frontend for easy visualization and CSV export

### Key Features
1. **Automatic Data Indexing**: Continuously indexes blockchain data from Ethereum nodes
2. **Historical Balance Tracking**: Stores daily validator balances for efficient querying
3. **MEV Rewards Tracking**: Identifies and tracks MEV rewards from relays and builders
4. **Withdrawal Tracking**: Monitors validator withdrawals (partial and full)
5. **Multi-Currency Support**: Provides rewards in multiple fiat currencies
6. **API Access**: RESTful API (v1 and v2) for programmatic access

---

## Architecture Overview

### System Components

The system consists of multiple independent services orchestrated via Docker Compose:

#### 1. **Frontend Services**
   - **frontend_vue**: Modern Vue3 web interface (primary)
   - **frontend_legacy**: Legacy frontend (TypeScript/Jinja2 templates)

#### 2. **API Service**
   - **api**: FastAPI-based REST API
   - Exposes endpoints for rewards, prices, and indexes
   - Includes rate limiting and Prometheus metrics
   - Multi-process deployment for high availability

#### 3. **Indexer Services** (Data Collection)
   - **indexer_balances**: Indexes validator balances daily
   - **indexer_block_rewards**: Indexes execution layer rewards (priority fees, MEV)
   - **indexer_validators**: Tracks validator lifecycle (activation, exit)
   - **indexer_withdrawals**: Indexes withdrawal events
   - **indexer_rocket_pool**: Rocket Pool specific data (minipool commissions, smoothing pool)
   - **indexer_prices**: Fetches and stores ETH price data from CoinGecko

#### 4. **Infrastructure Services**
   - **beacon_node**: Lighthouse consensus client (CL)
   - **geth**: Geth execution client (EL)
   - **db**: PostgreSQL database (stores all indexed data)
   - **redis**: Caching layer for API responses
   - **caddy**: Reverse proxy for HTTP/HTTPS
   - **prometheus**: Metrics collection
   - **grafana**: Monitoring dashboards
   - **adminer**: Database management UI

### Data Flow

```
Ethereum Blockchain
        ↓
   ┌────────────────────────────────────┐
   │  Beacon Node (Lighthouse)          │  ← Consensus Layer
   │  Geth (Execution Node)             │  ← Execution Layer
   └────────────────────────────────────┘
                ↓
   ┌────────────────────────────────────┐
   │      Indexer Services              │
   │  - balances.py                     │
   │  - block_rewards/main.py           │
   │  - validators.py                   │
   │  - withdrawals.py                  │
   │  - rocket_pool/main.py             │
   │  - prices.py                       │
   └────────────────────────────────────┘
                ↓
   ┌────────────────────────────────────┐
   │     PostgreSQL Database            │
   │  Tables:                           │
   │  - validator_balances_daily        │
   │  - block_rewards                   │
   │  - withdrawals                     │
   │  - eth_prices                      │
   │  - rocket_pool_*                   │
   └────────────────────────────────────┘
                ↓
   ┌────────────────────────────────────┐
   │         FastAPI                    │
   │  - /api/v1/rewards                 │
   │  - /api/v2/rewards                 │
   │  - /api/v2/prices                  │
   └────────────────────────────────────┘
                ↓
   ┌────────────────────────────────────┐
   │      Vue3 Frontend                 │
   │  - Validator input                 │
   │  - Date range selection            │
   │  - Rewards visualization           │
   │  - CSV export                      │
   └────────────────────────────────────┘
```

---

## Libraries and Dependencies

### Core Python Dependencies (from requirements.in)

| Library | Purpose |
|---------|---------|
| **aiofiles** | Async file I/O operations |
| **alembic** | Database migrations (SQLAlchemy) |
| **backoff** | Retry logic with exponential backoff |
| **fastapi** | Modern async web framework for API |
| **fastapi-plugins** | FastAPI plugins (Redis integration) |
| **fastapi-limiter** | Rate limiting for API endpoints |
| **httpx** | Async HTTP client for external API calls |
| **jinja2** | Template engine for legacy frontend |
| **psycopg2-binary** | PostgreSQL database adapter |
| **pytest** | Testing framework |
| **pytest-asyncio** | Async support for pytest |
| **pytz** | Timezone handling |
| **PyYAML** | YAML parsing for configuration |
| **requests** | HTTP library (fallback/sync operations) |
| **starlette_exporter** | Prometheus metrics for Starlette/FastAPI |
| **sqlalchemy** | SQL ORM and query builder |
| **tqdm** | Progress bars for indexing operations |
| **uvicorn[standard]** | ASGI server for FastAPI |
| **zstandard** | Compression library |

### External Services/APIs

1. **Beacon Node (Lighthouse)**
   - Consensus layer data
   - Validator balances, duties, and attestations
   - Historical state access via `--reconstruct-historic-states`

2. **Execution Node (Geth)**
   - Execution layer transaction data
   - Block receipts and logs
   - Smart contract interactions

3. **CoinGecko API**
   - Historical ETH price data
   - Multiple currency conversions

4. **MEV Relays**
   - MEV-Boost relay data
   - Block builder information
   - Proposer rewards

5. **Infura (Optional)**
   - Backup beacon node access
   - Archive execution node access

### Frontend Dependencies (Vue3)

- **Vue 3**: Progressive JavaScript framework
- **TypeScript**: Type-safe JavaScript
- **Vite**: Build tool and dev server
- **Chart.js/ApexCharts**: Data visualization (likely)

---

## Function Call Flow

### Entry Points

The system has multiple independent entry points that run as separate services:

1. **API Server**: `src/api/app.py`
2. **Balance Indexer**: `src/indexer/balances.py`
3. **Block Rewards Indexer**: `src/indexer/block_rewards/main.py`
4. **Validators Indexer**: `src/indexer/validators.py`
5. **Withdrawals Indexer**: `src/indexer/withdrawals.py`
6. **Rocket Pool Indexer**: `src/indexer/rocket_pool/main.py`
7. **Prices Indexer**: `src/indexer/prices.py`

### API Server Startup Flow

```
docker-compose.yml → tools/entrypoint_api_multiproc.sh → uvicorn → src/api/app.py
                                                                         ↓
                                                              FastAPI app creation
                                                                         ↓
                                                              Middleware setup:
                                                              - CORS
                                                              - Prometheus metrics
                                                                         ↓
                                                              Router registration:
                                                              - api_v1_router
                                                              - api_v2_router
                                                                         ↓
                                                              @app.on_event("startup")
                                                                         ↓
                                                              Initialize plugins:
                                                              - redis_plugin
                                                              - beacon_node_plugin
                                                              - coin_gecko_plugin
                                                              - db_plugin
                                                              - FastAPILimiter
```

### Indexer Startup Flow (Generic Pattern)

All indexers follow a similar pattern:

```
docker-compose.yml → python src/indexer/<name>.py
                              ↓
                    setup_logging()
                              ↓
                    Initialize providers:
                    - BeaconNodeProvider
                    - ExecutionNodeProvider
                    - DatabaseProvider
                              ↓
                    main() / async def run()
                              ↓
                    Continuous loop:
                    1. Fetch data from blockchain
                    2. Process/transform data
                    3. Write to database
                    4. Sleep/wait for next slot
```

---

## Detailed Function Descriptions

### Core Indexers

#### Balance Indexer (`src/indexer/balances.py`)

**Entry Point:** `index_balances()`

**Purpose:** Indexes validator balances at two critical points:
1. **Activation Slot**: When a validator first becomes active
2. **End-of-Day**: Daily snapshot at 23:59:59 UTC

**Flow:**
```
1. Calculate needed slots:
   - Activation slots for all validators
   - End-of-day slots from genesis to present
2. Remove already-indexed slots from DB
3. For each slot (oldest first):
   - Wait for slot finalization
   - Fetch all validator balances from beacon node
   - Insert into database (ON CONFLICT DO NOTHING)
   - Update Prometheus metrics
4. Sleep for 5 minutes, repeat
```

**Key Functions:**
- `beacon_node.balances_for_slot(slot, validator_indexes)`: Fetches validator balances
- `beacon_node.is_slot_finalized(slot)`: Ensures data stability
- Database uses bulk insert for efficiency

**Database Table:** `balance(validator_index, slot, balance)`

---

#### Block Rewards Indexer (`src/indexer/block_rewards/main.py`)

**Entry Point:** `index_block_rewards()`

**Purpose:** Indexes execution layer rewards including:
- Priority fees (tips)
- MEV rewards
- Block proposer information
- Fee recipient addresses

**Flow:**
```
1. Get range of slots to index (from first PoS slot to head finalized)
2. Remove already-indexed slots
3. Process slots concurrently (semaphore limit: 10)
4. For each slot:
   - Get proposer data (validator index, fee recipient, block number)
   - Call get_block_reward_value() for MEV detection
   - Store results in block_reward table
5. Sleep for 60 seconds, repeat
```

**MEV Detection Logic** (`block_rewards_mev_simple.py`):

The `get_block_reward_value()` function implements sophisticated MEV detection:

1. **Query MEV Relays**: Checks 9 major MEV relays for delivered payloads
   - Flashbots, Ultrasound, Agnostic, bloXroute, SecureRPC, etc.
   - Uses `asyncio.as_completed()` for first-response wins

2. **If relay match found**:
   - Extract expected MEV value and recipient
   - Verify fee recipient balance change matches expected value
   - Return MEV details

3. **If no relay match**:
   - Check block extra_data for relay indicators
   - Calculate fee recipient balance change
   - Compare to priority transaction fees
   - If balance change > tx fees → likely MEV
   - Check for transactions to known MEV bot contracts
   - Check for builder fee recipient patterns

4. **Balance Change Calculation** (`_get_balance_change_adjusted()`):
   ```
   balance_change = balance[block_n] - balance[block_n-1]

   Adjustments:
   + Smart contract reward distributions (Lido, RP, Stakefish, Kraken)
   + Outgoing transactions from fee recipient
   - Incoming withdrawals from beacon chain
   - Spam transactions (1 wei transfers from known spammers)
   ```

5. **Edge Cases Handled**:
   - Smart contract fee recipients (Rocket Pool fee distributors, Lido, etc.)
   - MEV bot contracts (79+ known addresses)
   - Builder fee recipients (70+ known builders)
   - Forwarder contracts (immediate distribution)
   - Manual inspection required for ambiguous cases

**Database Table:** `block_reward(slot, block_number, proposer_index, fee_recipient, priority_fees_wei, mev, mev_reward_recipient, mev_reward_value_wei, reward_processed_ok)`

---

#### Withdrawals Indexer (`src/indexer/withdrawals.py`)

**Purpose:** Indexes all validator withdrawals (partial and full) since Shapella upgrade

**Flow:**
```
1. Start from Shapella activation slot (6209536)
2. For each slot up to head finalized:
   - Fetch withdrawals from beacon node
   - Link to withdrawal address
   - Insert into database
3. Sleep and repeat
```

**Database Tables:**
- `withdrawal(id, slot, validator_index, amount_gwei, withdrawal_address_id)`
- `withdrawal_address(id, address)`

---

#### Rocket Pool Indexer (`src/indexer/rocket_pool/main.py`)

**Purpose:** Indexes Rocket Pool-specific data for node operators

**Flow:**
```
1. Index Nodes:
   - Query NodeRegistered events
   - Get fee distributor addresses
   - Store in rocket_pool_node table

2. Index Minipools:
   - Query MinipoolCreated events
   - Get validator pubkey, initial bond, initial fee
   - Store in rocket_pool_minipool table

3. Index Bond Reductions:
   - Query BondReduced events
   - Track new bond amounts and fees over time
   - Store in rocket_pool_bond_reduction table

4. Index Rewards:
   - Fetch reward snapshots from IPFS/GitHub
   - Parse merkle tree data
   - Store collateral RPL and smoothing pool ETH rewards
   - Store in rocket_pool_reward_period and rocket_pool_reward tables
```

**Database Tables:**
- `rocket_pool_node(node_address, fee_distributor)`
- `rocket_pool_minipool(minipool_address, validator_pubkey, initial_bond_value, initial_fee_value, node_address)`
- `rocket_pool_bond_reduction(minipool_address, timestamp, new_bond_amount, new_fee)`
- `rocket_pool_reward_period(reward_period_index, reward_period_end_time)`
- `rocket_pool_reward(node_address, reward_period_index, reward_collateral_rpl, reward_smoothing_pool_wei)`

---

#### Prices Indexer (`src/indexer/prices.py`)

**Purpose:** Indexes daily ETH and RPL prices from CoinGecko

**Flow:**
```
1. Determine date range (genesis to yesterday)
2. Query database for existing prices
3. Identify missing dates
4. For each missing date:
   - Call CoinGecko API
   - Get close price in all supported currencies
   - Insert into database
5. Sleep and repeat
```

**Database Table:** `price(token, currency, timestamp, value)`

---

#### Validators Indexer (`src/indexer/validators.py`)

**Purpose:** Indexes all validators and their public keys

**Flow:**
```
1. Fetch all validators from beacon node
2. For each validator:
   - Store validator_index and pubkey
   - ON CONFLICT DO NOTHING (idempotent)
```

**Database Table:** `validator(validator_index, pubkey)`

---

### API Endpoints Deep Dive

#### `/api/v2/rewards/full` - Full Validator Rewards

**Purpose:** Calculate comprehensive rewards for regular (non-Rocket Pool) validators

**Input:**
```json
{
  "validator_indexes": [1, 2, 3],
  "start_date": "2024-01-01",
  "end_date": "2024-12-31",
  "expected_fee_recipient_addresses": ["0x..."]
}
```

**Algorithm:**

1. **Get Initial Balances** (for each validator):
   ```
   IF validator activated after start_date:
       initial_balance = balance at activation slot
   ELSE:
       initial_balance = balance at start_date (00:00:00)
   ```

2. **Get End-of-Day Balances**:
   - Fetch balance for each day at 23:59:59 UTC
   - Range: start_date to end_date

3. **Get Withdrawals**:
   - Query all withdrawals in date range
   - Separate partial (<8 ETH) from full (>8 ETH) withdrawals

4. **Get Block Rewards**:
   - Query all blocks proposed by validators
   - Verify all were processed successfully
   - Validate fee recipients match expected addresses

5. **Calculate Consensus Layer Rewards** (per day, per validator):
   ```python
   # Balance delta
   amount_earned = (end_of_day_balance - previous_balance) * 1e18  # Convert to wei

   # Add back withdrawals that occurred during the day
   for withdrawal in withdrawals_this_day:
       if withdrawal.amount > 8 ETH:
           # Full withdrawal - only count rewards (amount - 32 ETH)
           amount_earned += (withdrawal.amount % 32_000_000_000) * 1e9
       else:
           # Partial withdrawal - count full amount
           amount_earned += withdrawal.amount * 1e9

   consensus_layer_rewards[date] = amount_earned
   ```

6. **Calculate Execution Layer Rewards** (per day, per validator):
   ```python
   for block_reward in blocks_proposed:
       date = slot_to_date(block_reward.slot)

       if block_reward.mev:
           reward = block_reward.mev_reward_value_wei
       else:
           reward = block_reward.priority_fees_wei

       execution_layer_rewards[date] += reward
   ```

**Output:**
```json
{
  "validator_rewards_list": [
    {
      "validator_index": 1,
      "consensus_layer_rewards": [
        {"date": "2024-01-01", "amount_wei": "1234567890123456"},
        ...
      ],
      "execution_layer_rewards": [
        {"date": "2024-02-15", "amount_wei": "9876543210987654"},
        ...
      ],
      "withdrawals": [
        {"date": "2024-03-01", "amount_wei": "2000000000000000000"},
        ...
      ]
    },
    ...
  ]
}
```

---

#### `/api/v2/rewards/rocket_pool` - Rocket Pool Rewards

**Purpose:** Calculate rewards for Rocket Pool node operators

**Key Differences from Full Endpoint:**

1. **Consensus Layer**: NOT calculated from balance changes
   - RP minipools have complex capital structures
   - User capital vs node capital makes balance tracking unreliable
   - CL rewards = `null` in response

2. **Withdrawals** (Node Operator Share):
   ```python
   # For each withdrawal, determine applicable bond & fee
   # (accounting for bond reductions over time)

   FULL_MINIPOOL_BOND = 32 ETH

   if withdrawal_date > bond_reduction_date:
       bond = new_bond_amount
       fee = new_fee
   else:
       bond = initial_bond_value
       fee = initial_fee_value

   if withdrawal < 8 ETH:
       # Partial withdrawal (rewards only)
       node_operator_share = withdrawal * (
           (bond / FULL_MINIPOOL_BOND) +                    # Node's capital portion
           ((FULL_MINIPOOL_BOND - bond) / FULL_MINIPOOL_BOND) * (fee / 1e18)  # Commission on user capital
       )
   else:
       # Full withdrawal (requires smart contract logic)
       # Calculate user capital, node capital, total rewards
       # Apply commission to rewards
       # See _get_rocket_pool_reward_share_withdrawal_for_bond_fee()
   ```

3. **Execution Layer** (Block Proposals):

   **If fee recipient = Smoothing Pool**:
   - Skip (accounted for in smoothing pool rewards)

   **If fee recipient = Fee Distributor**:
   ```python
   # Get balance change of fee distributor contract
   fd_balance_before = get_balance(fee_distributor, block_number - 1)
   fd_balance_after = get_balance(fee_distributor, block_number)
   fd_balance_change = fd_balance_after - fd_balance_before

   # Query fee distributor contract for node operator share
   try:
       node_share = fee_distributor.getNodeShare(block_number)
   except:
       # Pre-Atlas contracts don't have getNodeShare
       # Calculate manually with average fee and assumed 2:1 collateralization
       avg_fee = get_node_average_fee(node_address, block_number)
       collateral_ratio = 2 * 1e18

       node_balance_change = fd_balance_change * 1e18 / collateral_ratio
       user_balance_change = fd_balance_change - node_balance_change
       node_share = node_balance_change + user_balance_change * avg_fee / 1e18

   execution_layer_rewards[date] += node_share
   ```

4. **Smoothing Pool & RPL Rewards**:
   - Fetched from Rocket Pool reward snapshots (28-day cycles)
   - Distributed via merkle trees
   - Returned separately in response

**Output:**
```json
{
  "validator_rewards_list": [
    {
      "validator_index": 1,
      "consensus_layer_rewards": null,  // Not calculated for RP
      "execution_layer_rewards": [...],
      "withdrawals": [...]  // Node operator share only
    },
    ...
  ],
  "rocket_pool_node_rewards": [
    {
      "date": "2024-01-15",
      "node_address": "0x...",
      "amount_wei": "1000000000000000000",  // Smoothing pool ETH
      "amount_rpl": "100000000000000000000"  // Collateral RPL rewards
    },
    ...
  ]
}
```

---

### Provider Functions

#### BeaconNode Provider

**Key Functions:**

- `slot_for_datetime(dt)`: Converts datetime to slot number
  - Formula: `(dt - GENESIS_DATETIME).total_seconds() // 12`

- `datetime_for_slot(slot, timezone)`: Converts slot to datetime
  - Formula: `GENESIS_DATETIME + (slot * 12 seconds)`

- `balances_for_slot(slot, validator_indexes)`: Fetches balances via `/eth/v1/beacon/states/{slot}/validator_balances`

- `activation_slots_for_validators(validator_indexes)`: Returns activation slot for each validator

- `get_slot_proposer_data(slot)`: Returns `SlotProposerData(slot, proposer_index, fee_recipient, block_number, block_hash)`

- `indexes_for_eth1_address(address)`: Queries beaconcha.in API for validator indexes that deposited from ETH1 address

---

#### ExecutionNode Provider

**Key Functions:**

- `get_balance(address, block_number)`: Returns account balance in wei

- `get_block(block_number, verbose=False)`: Returns block data (optionally with full transactions)

- `get_block_receipts(block_number)`: Returns all transaction receipts (uses `eth_getBlockReceipts`)

- `get_block_priority_tx_fees(block_number, tx_fees_total)`:
  ```python
  burnt_fees = base_fee_per_gas * gas_used
  priority_fees = tx_fees_total - burnt_fees
  return priority_fees
  ```

- `get_logs(address, block_number_range, topics)`: Queries event logs with automatic range splitting for pagination

- `eth_call(params)`: Makes read-only smart contract calls

---

#### DbProvider

**Key Functions:**

- `balances(slots, validator_indexes)`: Queries balance table

- `block_rewards(min_slot, max_slot, proposer_indexes)`: Queries block_reward table

- `withdrawals(min_slot, max_slot, validator_indexes)`: Queries withdrawal table with joins

- `minipools_for_validators(validator_indexes)`: Returns Rocket Pool minipool objects

- `close_price_for_date(token, currency, date)`: Queries price table

All queries use SQLAlchemy ORM and are wrapped in Prometheus histogram timers.

---

### Database Schema Summary

**Core Tables:**
1. `balance` - Validator balances at activation + end-of-day
2. `block_reward` - Execution layer rewards (EL proposals, MEV)
3. `withdrawal` - All validator withdrawals
4. `withdrawal_address` - Withdrawal addresses
5. `validator` - Validator index to pubkey mapping
6. `price` - Historical token prices

**Rocket Pool Tables:**
7. `rocket_pool_node` - Node operators
8. `rocket_pool_minipool` - Minipools (validator contracts)
9. `rocket_pool_bond_reduction` - Bond reduction events
10. `rocket_pool_reward_period` - 28-day reward periods
11. `rocket_pool_reward` - Node rewards (smoothing pool + RPL)

**Relationships:**
- `withdrawal.withdrawal_address_id` → `withdrawal_address.id`
- `rocket_pool_minipool.node_address` → `rocket_pool_node.node_address`
- `rocket_pool_bond_reduction.minipool_address` → `rocket_pool_minipool.minipool_address`
- `rocket_pool_reward.reward_period_index` → `rocket_pool_reward_period.reward_period_index`

---

## Taxation Logic Overview

### Layman's Understanding of Ethereum Staking Taxation

**The Core Concept:**
When you run an Ethereum validator, you earn rewards in ETH. Most tax jurisdictions treat these rewards as **income** at the time you receive them. This means you need to report:

1. **How much ETH you earned** (in native currency)
2. **The fair market value** of that ETH on the day you earned it

**Example:**
- You earn 0.01 ETH on January 1, 2024
- ETH price on that day was $2,000
- You report $20 of income on January 1, 2024

### Types of Rewards Tracked

#### 1. Consensus Layer (CL) Rewards
**What it is:** Daily balance increases from validator duties
- Attesting to blocks
- Proposing blocks (without MEV)
- Sync committee participation

**How it's calculated:**
```
CL Reward = (Balance Today - Balance Yesterday) + Withdrawals Today
```

**Why withdrawals are added back:**
- If you withdraw 0.5 ETH, your balance decreases by 0.5 ETH
- But that 0.5 ETH was income you already earned
- So we add it back to show the true income for that day

**Tax Treatment:** Income on the date earned (daily)

---

#### 2. Execution Layer (EL) Rewards
**What it is:** Rewards from proposing blocks on the execution layer
- **Priority Fees (Tips)**: Users pay extra to get transactions included faster
- **MEV (Maximal Extractable Value)**: Block builders pay validators for the right to construct profitable blocks

**How it's calculated:**
- Query the `block_reward` table for blocks you proposed
- If MEV detected: use `mev_reward_value_wei`
- If no MEV: use `priority_fees_wei`

**Tax Treatment:** Income on the date the block was proposed

---

#### 3. Withdrawals
**What it is:** ETH moved from validator balance to execution layer address

**Types:**
- **Partial Withdrawal** (<8 ETH): Rewards skimmed automatically
- **Full Withdrawal** (≥32 ETH): Validator exit, includes principal + rewards

**Tax Treatment:**
- Partial: Already counted in CL rewards, tracking for record-keeping
- Full: Only the rewards portion (amount - 32 ETH) is income

**Important Note:** ethstaker.tax does NOT handle capital gains from selling ETH. It only tracks:
- **Income**: When you earn ETH from staking
- **Basis**: The fair market value when earned (for future capital gains calculations)

---

### Rocket Pool Specific Taxation

Rocket Pool node operators have additional complexities:

#### Node Operator Share Calculation

**The Problem:** A Rocket Pool minipool contains:
- Node operator capital (e.g., 8 ETH after bond reduction)
- User capital (e.g., 24 ETH from rETH holders)
- Total: 32 ETH

**The Solution:** Calculate node operator's share of rewards using:
```
Node Share = Node's capital portion + Commission on user's capital
```

**Example:**
- Bond: 8 ETH (25% of 32 ETH)
- Commission: 14% (0.14)
- Daily reward: 0.01 ETH

```
Node Share = 0.01 * (0.25 + 0.75 * 0.14)
           = 0.01 * (0.25 + 0.105)
           = 0.01 * 0.355
           = 0.00355 ETH
```

#### Bond Reductions Over Time

**The Challenge:** Rocket Pool allows reducing bond from 16 ETH → 8 ETH → (future) 4 ETH ETH

**The Solution:** Track bond reductions and apply correct bond/fee for each withdrawal's date
- Withdrawal on Jan 1 (before reduction): Use 16 ETH bond, old fee
- Withdrawal on Feb 1 (after reduction): Use 8 ETH bond, new fee

#### Smoothing Pool

**What it is:** Rocket Pool's optional shared execution layer rewards pool

**How it works:**
- Node operators opt in to send all EL rewards to smoothing pool
- Every 28 days, rewards are distributed based on performance
- More predictable income, reduces variance

**Tax Treatment:**
- Count as income on the reward period end date
- ethstaker.tax fetches this data from Rocket Pool's reward merkle trees

#### RPL Collateral Rewards

**What it is:** Rocket Pool token rewards for maintaining RPL collateral

**Tax Treatment:** Income on reward period end date (in RPL, converted to currency value)

---

### Key Tax Concepts Implemented

#### 1. Accrual Basis Accounting
Rewards are reported when **earned**, not when withdrawn or sold

#### 2. Fair Market Value (FMV)
ETH price from CoinGecko on the date of earning determines income value

#### 3. Multiple Currency Support
Report income in your tax jurisdiction's currency (USD, EUR, GBP, etc.)

#### 4. Timezone Support
End-of-day calculations use UTC by default (API v1 supports other timezones)

#### 5. CSV Export
Generate CSV files compatible with tax software or accountants

---

### What ethstaker.tax Does NOT Handle

1. **Capital Gains:** When you sell ETH (consult a tax professional)
2. **Slashing:** Penalties are not income, may be deductible losses
3. **Validator Setup Costs:** Hardware, electricity (may be deductible)
4. **Entity Structure:** Personal vs. business taxation
5. **Tax Filing:** You still need to file taxes yourself or hire an accountant

---

## GitHub Issues Analysis

Based on recent GitHub issues, here are the main categories and surface areas:

### 1. **Execution Layer Rewards Not Available** (Issue #116, #113)
**Surface Area:** Block rewards indexer, MEV detection

**Problem:** Some blocks fail to process, showing "500 execution layer rewards not available"

**Root Causes:**
- MEV detection edge cases not covered
- Balance change calculations fail for complex smart contract interactions
- Relays down or not responding
- Missing data for historical blocks

**Code Locations:**
- `src/indexer/block_rewards/block_rewards_mev_simple.py`: MEV detection logic
- `block_reward.reward_processed_ok = False`: Marks failed blocks

**Impact:** Users cannot generate reports for date ranges containing failed blocks

**Mitigation:** Code raises `ManualInspectionRequired` exception, stores failure in database

---

### 2. **EIP-7251 (MaxEB) Support** (Issue #115)
**Surface Area:** Consensus layer reward calculation, validator consolidation

**Problem:** Validator consolidation (combining multiple 32 ETH validators into one large validator) causes incorrect income calculations

**Root Cause:**
- Balance transfers between validators not tracked
- Consolidated validators show unrealistically large CL rewards
- System counts transferred balance as "earned" income

**Workaround:** User reports disabling "Consensus layer income upon withdrawal" fixes display

**Long-term Fix Needed:**
1. Track consolidation events (`0x02` validator type)
2. Detect balance transfers between validators
3. Exclude transfers from income calculations
4. Only count actual staking rewards

**Code Locations:**
- `src/api/api_v2/endpoints/rewards.py`: CL reward calculation (lines 527-554)
- `src/indexer/validators.py`: Would need to track validator types

---

### 3. **Currency Retrieval Failures** (Issue #114, #112)
**Surface Area:** CoinGecko API integration

**Problem:** "Failed to retrieve supported currencies" error on page load

**Root Causes:**
- CoinGecko API rate limiting
- CoinGecko API downtime
- Network issues

**Code Locations:**
- `src/providers/coin_gecko.py`: CoinGecko API calls
- `src/api/api_v1/endpoints/currencies.py`: `/supported_currencies` endpoint

**Impact:** Users cannot access the frontend

**Mitigation:** Implement:
- Fallback currency list
- Longer cache TTL for currencies
- Retry logic with backoff (already exists for some calls)

---

### 4. **Partial Withdrawal Handling** (Issue #113)
**Surface Area:** Full withdrawal detection, Pectra upgrade

**Problem:** "Full withdrawal for X with <32 ETH leads to negative income"

**Root Cause:**
- After Pectra, some withdrawals >8 ETH but <32 ETH
- System incorrectly classifies as full withdrawal
- Calculation: `(withdrawal.amount % 32 ETH) * 1e9` produces incorrect value

**Code Location:**
- `src/api/api_v2/endpoints/rewards.py:535-540`

**Fix Needed:**
- Update full withdrawal detection logic
- Account for MaxEB validators with >32 ETH balances
- Differentiate between:
  - Full exit with <32 ETH (slashed)
  - Partial withdrawal from MaxEB validator
  - Full exit from MaxEB validator

---

### 5. **Surface Area by Component**

#### High-Touch Areas (Frequent Changes Needed):
1. **MEV Detection** (`block_rewards_mev_simple.py`):
   - New MEV builders emerge
   - New relay patterns
   - New smart contract forwarders
   - Estimated: 10-20 edge cases per year

2. **Rocket Pool Integration** (`rocket_pool/main.py`, `rewards.py`):
   - Protocol upgrades (Atlas, Saturn, etc.)
   - New smart contract versions
   - Bond/fee structure changes
   - Estimated: 2-4 updates per year

3. **Beacon Chain Upgrades** (Multiple files):
   - Shapella (withdrawals): ✅ Implemented
   - Pectra (MaxEB): ❌ Not yet implemented
   - Future: ePBS, etc.
   - Estimated: 1-2 major upgrades per year

#### Medium-Touch Areas:
4. **API Endpoints** (`api/api_v*/endpoints/*.py`):
   - Performance optimizations
   - New features
   - Bug fixes
   - Estimated: 5-10 updates per year

5. **Database Schema** (`db/tables.py`, `alembic/versions/`):
   - New data types (validator types, etc.)
   - Performance indexes
   - Estimated: 2-4 migrations per year

#### Low-Touch Areas:
6. **Core Indexers** (`indexer/balances.py`, `validators.py`, etc.):
   - Stable, rarely change
   - Estimated: 0-2 updates per year

7. **Frontend** (`frontend_vue/`):
   - UI improvements
   - New features
   - Estimated: Variable, depends on priorities

---

### Issue Patterns and Recommendations

**Pattern 1: External API Dependencies**
- CoinGecko, Infura, MEV relays
- **Recommendation:** Implement circuit breakers, fallbacks, aggressive caching

**Pattern 2: Protocol Upgrade Lag**
- Pectra released, ethstaker.tax not yet updated
- **Recommendation:** Monitor Ethereum roadmap, plan updates in advance

**Pattern 3: Edge Case Accumulation**
- MEV detection has 100+ lines of special cases
- **Recommendation:** Refactor into pattern-based system, add comprehensive tests

**Pattern 4: User Error Messages**
- Generic "500 error" messages
- **Recommendation:** Improve error messages, provide actionable guidance

---

## Adding Network Upgrade Support

### Example 1: Adding Support for Pectra (EIP-7251 MaxEB)

**What Changed in Pectra:**
- Validators can now have >32 ETH (up to 2048 ETH effective balance)
- New validator type: `0x02` (compounding withdrawal credentials)
- Validator consolidation: Combining multiple validators into one
- Partial withdrawals from >32 ETH validators

**Required Code Changes:**

#### 1. Database Schema Updates

**Add validator type tracking** (`src/db/tables.py`):
```python
class Validator(Base):
    __tablename__ = "validator"

    validator_index = Column(Integer, nullable=False, primary_key=True)
    pubkey = Column(String(length=98), nullable=False, index=True)
    withdrawal_credentials = Column(String(length=98), nullable=True)  # NEW
    effective_balance_cap = Column(Integer, nullable=True)  # NEW (32, 2048, etc.)
```

**Create migration:**
```bash
make migration-generate MIGRATION_NAME="add_validator_type_tracking"
```

---

#### 2. Track Consolidation Events

**New indexer** (`src/indexer/consolidations.py`):
```python
async def index_consolidations():
    """
    Track validator consolidation events.
    Prevents double-counting balance transfers as income.
    """
    # Query beacon node for consolidation requests
    # Store: source_validator, target_validator, slot, amount
    # Database table: consolidation(source_index, target_index, slot, amount_gwei)
```

**Database table:**
```python
class Consolidation(Base):
    __tablename__ = "consolidation"

    id = Column(Integer, primary_key=True)
    source_validator_index = Column(Integer, nullable=False, index=True)
    target_validator_index = Column(Integer, nullable=False, index=True)
    slot = Column(Integer, nullable=False)
    amount_gwei = Column(Numeric(precision=18), nullable=False)
```

---

#### 3. Update Reward Calculation Logic

**Modify** (`src/api/api_v2/endpoints/rewards.py:527-554`):
```python
# OLD:
amount_earned_wei = Decimal(1e18) * (eod_balance.balance - prev_balance.balance)
amount_earned_wei += amount_withdrawn_this_day_wei

# NEW:
amount_earned_wei = Decimal(1e18) * (eod_balance.balance - prev_balance.balance)
amount_earned_wei += amount_withdrawn_this_day_wei

# Subtract consolidation transfers IN (balance increased but not income)
for consolidation in consolidations_received_this_day:
    amount_earned_wei -= consolidation.amount_gwei * Decimal(1e9)

# ADD consolidation transfers OUT (balance decreased but rewards still earned)
for consolidation in consolidations_sent_this_day:
    amount_earned_wei += consolidation.amount_gwei * Decimal(1e9)
```

---

#### 4. Update Full Withdrawal Detection

**Modify** (`src/api/api_v2/endpoints/rewards.py:533-543`):
```python
# OLD:
if w.amount_gwei > 8 * Decimal(1e9):
    # Assume full withdrawal
    if w.amount_gwei < 32 * Decimal(1e9):
        raise HTTPException(...)
    amount_withdrawn_this_day_wei += (w.amount_gwei % (32 * Decimal(1e9))) * Decimal(1e9)

# NEW:
validator_effective_balance = db_provider.get_validator_effective_balance(validator_index)

if w.amount_gwei > 8 * Decimal(1e9):
    # Could be full withdrawal OR large partial from MaxEB validator
    validator_balance = db_provider.balances(slots=[w.slot], validator_indexes=[validator_index])[0].balance

    if validator_balance < 1:  # Validator exited
        # Full withdrawal - only count rewards
        principal = Decimal(32)  # or use original deposit amount if tracked
        amount_withdrawn_this_day_wei += max(0, (w.amount_gwei * Decimal(1e9) - principal * Decimal(1e18)))
    else:
        # Large partial withdrawal from MaxEB validator
        amount_withdrawn_this_day_wei += w.amount_gwei * Decimal(1e9)
```

---

#### 5. Update Validator Indexer

**Modify** (`src/indexer/validators.py`):
```python
async def index_validators():
    validators = await beacon_node.get_validators("head")

    for validator in validators["data"]:
        session.merge(
            Validator(
                validator_index=validator["index"],
                pubkey=validator["validator"]["pubkey"],
                withdrawal_credentials=validator["validator"]["withdrawal_credentials"],  # NEW
                effective_balance_cap=get_effective_balance_cap(
                    validator["validator"]["withdrawal_credentials"]
                ),  # NEW
            )
        )

def get_effective_balance_cap(withdrawal_credentials: str) -> int:
    """Determine max effective balance based on withdrawal credentials."""
    if withdrawal_credentials.startswith("0x02"):
        return 2048  # MaxEB validator
    else:
        return 32  # Legacy validator
```

---

#### 6. Add Docker Compose Service

**Modify** (`docker-compose.yml`):
```yaml
  indexer_consolidations:
    image: eth2-tax:latest
    restart: unless-stopped
    command: [ "python", "./src/indexer/consolidations.py" ]
    environment:
      DB_URI:
      BEACON_NODE_HOST:
      BEACON_NODE_PORT:
    depends_on:
      - db
```

---

### Example 2: Adding Support for ePBS (Enshrined Proposer-Builder Separation)

**What Changes in ePBS:**
- Block proposal and building become protocol-level
- New transaction types for builder bids
- Execution layer rewards flow changes

**Required Code Changes:**

#### 1. Update MEV Detection Logic

Since ePBS makes MEV extraction protocol-level, the current relay-based detection becomes obsolete.

**Modify** (`src/indexer/block_rewards/block_rewards_mev_simple.py`):
```python
async def get_block_reward_value_epbs(
    slot_proposer_data: SlotProposerData,
    execution_node: ExecutionNode,
) -> BlockRewardValue:
    """
    For ePBS blocks, MEV reward is part of the protocol.
    Query execution payload for builder bid amount.
    """
    block = await execution_node.get_block(slot_proposer_data.block_number)

    # ePBS blocks have a new field: builder_bid_value
    if "builder_bid_value" in block:
        return BlockRewardValue(
            block_priority_tx_fees=0,  # No longer separate
            contains_mev=True,
            mev_recipient=slot_proposer_data.fee_recipient,
            mev_recipient_balance_change=int(block["builder_bid_value"], 16)
        )

    # Pre-ePBS logic fallback
    return await get_block_reward_value(...)
```

#### 2. Add ePBS Activation Detection

```python
EPBS_ACTIVATION_SLOT = 999999999  # Update when known

async def process_slot(slot: int):
    if slot >= EPBS_ACTIVATION_SLOT:
        block_reward_value = await get_block_reward_value_epbs(...)
    else:
        block_reward_value = await get_block_reward_value(...)
```

---

### Example 3: Adding Rocket Pool-Specific Features (e.g., New Bond Sizes)

**What Changes:**
- Rocket Pool governance approves 4 ETH bond minipools
- New minipool factory contract

**Required Code Changes:**

#### 1. Update Rocket Pool Constants

**Modify** (`src/providers/rocket_pool.py`):
```python
# Add new minipool factory address
MINIPOOL_FACTORY_ADDRESSES = {
    "0x...": "legacy",
    "0x...": "atlas",
    "0x...": "4eth_bond",  # NEW
}
```

#### 2. Update Minipool Creation Indexing

**Modify** (`src/indexer/rocket_pool/main.py`):
```python
# Add new event signature for 4 ETH minipool creation
MINIPOOL_CREATED_TOPICS = [
    "0x...",  # Legacy signature
    "0x...",  # Atlas signature
    "0x...",  # 4 ETH bond signature (NEW)
]
```

#### 3. Test Reward Share Calculations

4 ETH bond means 12.5% capital share (4/32):
```python
# Node share formula still works:
node_share = reward * (
    (4 / 32) +  # Node capital: 12.5%
    (28 / 32) * (commission / 1e18)  # User capital: 87.5% * commission
)
```

No code changes needed if formula is generic!

---

### General Upgrade Process

1. **Monitor Ethereum Roadmap**
   - Subscribe to: ethereum/pm, ethereum/consensus-specs
   - Track EIP status: eips.ethereum.org

2. **Identify Impact**
   - Does it change validator rewards?
   - Does it change block rewards?
   - Does it add new data structures?

3. **Update Schema**
   - Add new database tables/columns
   - Run migrations

4. **Update Indexers**
   - Add new indexer services if needed
   - Update existing indexers for new data formats

5. **Update Calculation Logic**
   - API endpoints may need new formulas
   - Test extensively with testnet data

6. **Update Frontend**
   - Display new data types
   - Update documentation

7. **Deploy**
   - Test on staging
   - Monitor metrics
   - Rollback plan ready

---

### Testing Strategy

#### Unit Tests
```python
# tests/test_rewards.py
def test_consolidation_reward_calculation():
    """Test that consolidation transfers don't count as income."""
    # Setup: validator receives 10 ETH consolidation + earns 0.01 ETH rewards
    # Assert: income = 0.01 ETH, not 10.01 ETH
```

#### Integration Tests
```python
# tests/test_pectra_integration.py
@pytest.mark.asyncio
async def test_maxeb_validator_rewards():
    """Test MaxEB validator reward tracking end-to-end."""
    # Index balances for MaxEB validator
    # Calculate rewards
    # Assert correct values
```

#### Testnet Validation
- Deploy to Holesky/Sepolia testnet
- Process real Pectra-activated validators
- Compare results with beaconcha.in

---

### Deployment Checklist

- [ ] Database migration tested
- [ ] Indexers updated and tested
- [ ] API endpoints return correct data
- [ ] Frontend displays new data
- [ ] Prometheus metrics added
- [ ] Documentation updated
- [ ] Rollback procedure documented
- [ ] Monitoring alerts configured

---

## CL+EL Stack Requirements

### Current Stack (from docker-compose.yml)

**Consensus Layer (CL):**
- **Client:** Lighthouse v7.0.1
- **Mode:** `--reconstruct-historic-states` (Archive mode for recent data)
- **Checkpoint Sync:** Yes (`--checkpoint-sync-url`)
- **Storage:** ~670GB+ (grows over time)

**Execution Layer (EL):**
- **Client:** Geth v1.15.11
- **Mode:** Full node with history expiry
- **Storage:** Variable (with history expiry enabled)

**JWT Authentication:** Shared secret for CL-EL communication

---

### Why ethstaker.tax Needs Its Own Node Stack

Unlike many applications that can rely on third-party RPC providers, ethstaker.tax has unique requirements:

#### 1. **Archive-Level Beacon Chain Data**
- Needs historical validator balances (not just current state)
- Requires balance lookups for arbitrary past slots
- Most public APIs don't provide this depth

#### 2. **High Request Volume**
- Indexing 1M+ validators daily
- Hundreds of balance queries per minute
- Would quickly hit rate limits on public APIs

#### 3. **Execution Layer Historical Data**
- Needs `eth_getBlockReceipts` for all post-merge blocks
- Requires balance lookups at specific block numbers
- Smart contract log queries for Rocket Pool data

#### 4. **Reliability**
- Taxation data must be accurate and auditable
- Can't tolerate API downtime or rate limiting
- Must have full control over data quality

---

### Understanding the Quote from the Maintainer

> "The CL+EL stack is actually part of the ethstaker.tax stack, you can see it here [docker-compose.yml]. I'm personally not a big fan of eth-docker due to the things it does in the background automagically. I prefer to know exactly which flags are being set for the clients at all times..."

**Translation:**
- ethstaker.tax runs its own Ethereum nodes (not relying on external services)
- `docker-compose.yml` explicitly defines all client configurations
- Prefers transparency over convenience (explicit flags vs. automated scripts)

> "In terms of client preferences, the future EL client I don't care that much about since I want to use an external RPC provider for EL data, so any EL that supports history expiry should be fine."

**Translation:**
- Plan to use external archive RPC (like Infura) for execution layer
- Local EL node only needs basic functionality + history expiry
- History expiry: Prune old blocks to save disk space (keep only recent X months)

**Current Implementation:**
- `EXECUTION_NODE_INFURA_ARCHIVE_URL`: Already configured
- `execution_node.get_balance(..., use_infura=True)`: Fallback to Infura for historical data

> "The CL client choice will be more interesting. ethstaker.tax requires "archive"-level data for recent epochs (up to a few days in case of downtime). Most CL clients in archive mode sync data all the way back to genesis which takes up a lot of space (current Lighthouse archive node takes up 670GB)."

**The Problem:**
- **Need**: Archive data for last ~2 weeks (to handle downtime/reindex)
- **Reality**: Most CL clients store archive data all the way to genesis
- **Result**: 670GB+ storage requirement (and growing)

> "I'd like to try using a CL client that supports a limited archive mode - pruning archive data older than, say, somewhere between 2-10 weeks. Hopefully that allows to run the entire stack on <2TB of storage long-term."

**The Goal:**
- **Limited Archive Mode**: Keep detailed state for recent ~2-10 weeks
- **Pruned Data**: Discard old detailed state (keep only headers/roots)
- **Storage Target**: <2TB total for entire stack

**Why This Matters:**
- Current growth: ~10-20GB/month
- In 5 years: 600GB → 1.2TB+
- Limited archive mode: Stable ~200-400GB

> "The Erigon+Caplin archive node only applies to EL data so it doesn't tell the whole story...I don't even know if Caplin has an archive mode. Also, we've faced some instability with Erigon so it's not going to be my preferred choice."

**Translation:**
- **Erigon**: Alternative EL client (good for archive storage efficiency)
- **Caplin**: Erigon's integrated CL client (experimental)
- **Issue**: Erigon had instability issues, not reliable enough for production
- **Uncertainty**: Caplin's archive capabilities unknown

---

### CL Client Comparison for ethstaker.tax

| Client | Archive Mode | Limited Archive | Storage | Stability | Notes |
|--------|-------------|----------------|---------|-----------|-------|
| **Lighthouse** | ✅ Yes | ❌ No | 670GB+ | ✅ Excellent | Current choice |
| **Prysm** | ✅ Yes | ❌ No | ~600GB+ | ✅ Excellent | Similar to Lighthouse |
| **Teku** | ✅ Yes | ⚠️ Partial | ~500GB+ | ✅ Good | Has some pruning options |
| **Nimbus** | ⚠️ Limited | ✅ Yes? | ~300GB+ | ✅ Good | Smaller footprint, investigate |
| **Lodestar** | ✅ Yes | ❓ Unknown | ~600GB+ | ⚠️ Good | TypeScript, slower |

**Future Investigation:**
- **Nimbus**: Known for efficiency, may have limited archive features
- **Teku**: Check `--data-storage-mode=prune` options
- **Custom Pruning**: Modify Lighthouse to add pruning (contribution back to ecosystem?)

---

### Recommended Stack Configurations

#### Option 1: Current Production Stack (Stable)
```yaml
CL: Lighthouse v7+ (archive mode)
EL: Geth v1.14+ (history expiry enabled)
Storage: 1.5TB+ recommended
Pros: Battle-tested, stable
Cons: High storage requirement
```

#### Option 2: Efficiency-Focused (Experimental)
```yaml
CL: Nimbus or Teku (limited archive)
EL: Nethermind (efficient pruning)
Storage: <1TB target
Pros: Lower storage costs
Cons: Needs testing, may lack features
```

#### Option 3: Hybrid (Recommended for New Deployments)
```yaml
CL: Lighthouse (self-hosted for recent data)
EL: Minimal local node + Infura/Alchemy archive
Storage: ~800GB
Pros: Balance of reliability and cost
Cons: Dependency on external EL provider
```

---

### Key Flags and Configuration

#### Lighthouse CL (Current)
```bash
lighthouse bn \
  --network mainnet \
  --http \
  --http-address 0.0.0.0 \
  --execution-endpoint http://geth:8551 \
  --execution-jwt /tmp/jwt_secret \
  --checkpoint-sync-url https://mainnet.checkpoint.sigp.io \
  --reconstruct-historic-states \     # CRITICAL: Enables balance lookups
  --disable-backfill-rate-limiting    # Speed up initial sync
```

**Key Flag Explanations:**
- `--reconstruct-historic-states`: Allows querying balances at arbitrary past slots
- `--checkpoint-sync-url`: Fast sync from recent checkpoint (hours vs. days)
- `--disable-backfill-rate-limiting`: Speeds up historical data download

#### Geth EL (Current)
```bash
geth \
  --datadir /data \
  --http \
  --http.addr 0.0.0.0 \
  --http.vhosts=* \
  --authrpc.jwtsecret /tmp/jwt_secret \
  --syncmode=snap                     # Fast sync mode
  # History expiry would add:
  # --history.transactions=90000      # Keep ~90 days of tx history
```

**Future Addition for History Expiry:**
```bash
--history.transactions=90000   # Keep 90 days
--history.state=180000         # Keep 180 days of state
```

---

### Storage Growth Projections

**Current Growth Rate:**
- CL Archive: ~15GB/month
- EL Full: ~5-10GB/month (with history expiry)
- Total: ~20-25GB/month

**5-Year Projections:**
| Year | CL Size | EL Size | Total | Notes |
|------|---------|---------|-------|-------|
| 2024 | 670GB | 300GB | ~1TB | Current |
| 2025 | 850GB | 360GB | ~1.2TB | +20% validators |
| 2026 | 1040GB | 420GB | ~1.5TB | Continued growth |
| 2027 | 1250GB | 480GB | ~1.8TB | Approaching 2TB |
| 2028 | 1470GB | 540GB | ~2.0TB | Hitting limit |

**With Limited Archive (Target):**
| Year | CL Size | EL Size | Total | Notes |
|------|---------|---------|-------|-------|
| 2024 | 300GB | 300GB | 600GB | After pruning |
| 2025+ | ~300-350GB | ~300GB | ~650GB | Stable |

---

### Infrastructure Recommendations

#### Hardware Requirements
**Minimum (Current Stack):**
- CPU: 4 cores / 8 threads
- RAM: 16GB
- Storage: 2TB NVMe SSD
- Network: 1Gbps

**Recommended (Production):**
- CPU: 8 cores / 16 threads (Ryzen 5600X or better)
- RAM: 32GB
- Storage: 4TB NVMe SSD (future-proof)
- Network: 1Gbps+ with good peering

**Cloud Costs (Estimated Monthly):**
- AWS m5.2xlarge + 2TB GP3: ~$300-400/month
- Hetzner dedicated (AX101): ~€120/month (~$130)
- Self-hosted: ~$1000 hardware + electricity

---

### Alternative: External RPC Providers

**If Not Running Own Stack:**

**Pros:**
- No infrastructure management
- No storage concerns
- Instant access

**Cons:**
- Monthly costs: $500-2000+ depending on usage
- Rate limits
- Less control over data quality
- Privacy concerns (query patterns reveal validator ownership)

**Providers:**
- **Infura**: Good reliability, expensive at scale
- **Alchemy**: Similar to Infura
- **QuickNode**: More expensive, good performance
- **Self-Hosted + Infura Fallback**: Best of both worlds (current plan)

---

### Action Items for New Maintainers

1. **Short-term (Current Stack)**:
   - Monitor storage growth
   - Set up alerts at 80% capacity
   - Budget for storage expansion

2. **Medium-term (Optimization)**:
   - Test Nimbus/Teku limited archive capabilities
   - Benchmark performance vs. storage tradeoffs
   - Consider migration if viable

3. **Long-term (Scalability)**:
   - Contribute limited archive mode to Lighthouse
   - Build pruning tools if needed
   - Consider sharding data across multiple nodes if user base grows

---

### Monitoring Key Metrics

**Node Health:**
```
beacon_head_slot - Current head slot
beacon_finalized_epoch - Last finalized epoch
sync_eth1_connected - EL connection status
```

**Storage:**
```
df -h /path/to/lighthouse
df -h /path/to/geth
```

**Indexer Health:**
```
slots_with_missing_balances - Backlog of balance indexing
slots_with_missing_block_rewards - Backlog of block reward indexing
beacon_node_request_count - Request volume to CL
execution_node_request_count - Request volume to EL
```

**Performance:**
```
db_query_duration_seconds - Database performance
http_request_duration_seconds - API response times
```

---

*This concludes the comprehensive documentation for ethstaker.tax!*

---

## API Code Deep Dive

*[This section will be populated with API analysis in the next commit]*

---

*Documentation in progress - this is a living document that will be updated as the codebase is analyzed further.*
