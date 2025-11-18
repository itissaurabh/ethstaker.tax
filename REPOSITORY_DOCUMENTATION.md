# ETHStaker.tax - Comprehensive Repository Documentation

## Table of Contents
1. [What the Repository Does](#what-the-repository-does)
2. [Architecture Overview](#architecture-overview)
3. [Dependencies and Libraries](#dependencies-and-libraries)
4. [Function Call Flow](#function-call-flow)
5. [Detailed Function Documentation](#detailed-function-documentation)
6. [Taxation Logic Overview](#taxation-logic-overview)
7. [GitHub Issues Analysis](#github-issues-analysis)
8. [Adding Network Upgrade Support](#adding-network-upgrade-support)
9. [CL+EL Stack Details](#clel-stack-details)
10. [API Documentation](#api-documentation)

---

## What the Repository Does

**ethstaker.tax** is a tax calculation tool for Ethereum stakers. It helps validators accurately calculate their taxable income from staking activities on the Ethereum blockchain.

### Core Functionality

1. **Validator Tracking**: Indexes and tracks all validators on the Ethereum network
2. **Balance Monitoring**: Records validator balances at end-of-day and activation points
3. **Reward Calculation**: Computes consensus layer (CL) and execution layer (EL) rewards
4. **Withdrawal Tracking**: Monitors both partial and full withdrawals from validators
5. **MEV Detection**: Identifies and calculates Maximum Extractable Value (MEV) rewards
6. **Rocket Pool Support**: Special handling for Rocket Pool validators with bond reductions and fee sharing
7. **Tax Reporting**: Generates tax reports with accurate income calculations in multiple currencies
8. **Price Data**: Fetches historical ETH prices for fiat currency conversions

### Key Features

- **Dual-mode operation**: Supports both solo stakers and Rocket Pool node operators
- **Historical data**: Stores complete historical balance and reward data
- **MEV tracking**: Detects and properly attributes MEV rewards
- **Multi-currency support**: Converts ETH rewards to various fiat currencies
- **REST API**: Provides programmatic access to all data
- **Real-time indexing**: Continuously indexes new blockchain data
- **Finality awareness**: Only processes finalized blockchain data

---

## Architecture Overview

### System Components

The system uses a **microservices architecture** with the following components:

```
┌─────────────────┐
│   Frontend      │ (Vue 3)
│   (Port 443)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   API Server    │ (FastAPI)
│   (Port 8000)   │
└────────┬────────┘
         │
         ├──────────────┬──────────────┬──────────────┐
         ▼              ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Indexer    │ │   Indexer    │ │   Indexer    │ │   Indexer    │
│  (Balances)  │ │  (Rewards)   │ │(Withdrawals) │ │  (Prices)    │
└──────┬───────┘ └──────┬───────┘ └──────┬───────┘ └──────┬───────┘
       │                │                │                │
       └────────────────┴────────────────┴────────────────┘
                        │
                        ▼
              ┌──────────────────┐
              │   PostgreSQL DB  │
              └──────────────────┘

       ┌────────────────┴────────────────┐
       ▼                                 ▼
┌──────────────┐                  ┌──────────────┐
│ Beacon Node  │                  │ Execution    │
│ (Lighthouse) │                  │ Node (Geth)  │
└──────────────┘                  └──────────────┘
```

### Docker Services

All services run in Docker containers (see `docker-compose.yml`):

1. **frontend_legacy**: Legacy frontend (being phased out)
2. **api**: FastAPI REST API server (multi-process via gunicorn)
3. **indexer_balances**: Indexes validator balances daily
4. **indexer_block_rewards**: Indexes execution layer rewards and MEV
5. **indexer_validators**: Indexes validator metadata
6. **indexer_withdrawals**: Indexes withdrawal events
7. **indexer_rocket_pool**: Indexes Rocket Pool-specific data
8. **indexer_prices**: Fetches ETH/fiat price data from CoinGecko
9. **beacon_node**: Lighthouse consensus client
10. **geth**: Go-Ethereum execution client
11. **db**: PostgreSQL database
12. **redis**: Cache and rate limiting
13. **caddy**: Reverse proxy and HTTPS termination
14. **prometheus**: Metrics collection
15. **grafana**: Metrics visualization
16. **adminer**: Database administration UI

---

## Dependencies and Libraries

### Core Python Dependencies

From `requirements.in`:

| Library | Purpose |
|---------|---------|
| **FastAPI** | Modern async web framework for building the REST API |
| **uvicorn[standard]** | ASGI server to run FastAPI |
| **SQLAlchemy** | ORM for database interactions |
| **alembic** | Database migration tool |
| **psycopg2-binary** | PostgreSQL adapter for Python |
| **httpx** | Async HTTP client for API calls to beacon/execution nodes |
| **backoff** | Retry logic with exponential backoff |
| **redis** | Cache and rate limiting |
| **fastapi-plugins** | Redis plugin for FastAPI |
| **fastapi-limiter** | Rate limiting middleware |
| **starlette_exporter** | Prometheus metrics exporter |
| **pytz** | Timezone handling |
| **requests** | HTTP library for synchronous calls |
| **aiofiles** | Async file operations |
| **jinja2** | Template engine |
| **PyYAML** | YAML parsing for config files |
| **tqdm** | Progress bars |
| **zstandard** | Compression for Rocket Pool data |
| **pytest** | Testing framework |
| **pytest-asyncio** | Async test support |

### External Services

- **Infura**: Backup Ethereum node provider (beacon + execution)
- **CoinGecko**: Cryptocurrency price data API
- **Beaconcha.in**: Validator lookup by ETH1 address

### Infrastructure

- **PostgreSQL 13.9**: Relational database
- **Redis 7.0.8**: In-memory cache
- **Lighthouse v7.0.1**: Consensus layer client
- **Geth v1.15.11**: Execution layer client
- **Caddy 2.6.2**: Web server and reverse proxy
- **Prometheus**: Metrics collection
- **Grafana**: Metrics visualization

---

## Function Call Flow

### Main Entry Points

The application has **8 independent processes** that run continuously:

#### 1. API Server (`src/api/app.py`)

**Entry Point**: `uvicorn src.api.app:app`

**Flow**:
```
app.py:on_startup()
  └─> setup_logging()
  └─> redis_plugin.init()
  └─> beacon_node_plugin.init_app()
  └─> coin_gecko_plugin.init_app()
  └─> db_plugin.init_app()
  └─> FastAPILimiter.init()

[User Request] → FastAPI Router
  ├─> /api/v1/rewards → api_v1.endpoints.rewards.rewards()
  ├─> /api/v2/rewards/full → api_v2.endpoints.rewards.rewards()
  ├─> /api/v2/rewards/rocket_pool → api_v2.endpoints.rewards.rewards()
  └─> /api/v2/prices → api_v2.endpoints.prices.prices()
```

#### 2. Balance Indexer (`src/indexer/balances.py`)

**Entry Point**: `python src/indexer/balances.py`

**Flow**:
```
while True:
  index_balances()
    ├─> beacon_node.activation_slots_for_validators()
    ├─> Calculate end-of-day slots for all dates
    ├─> Remove already indexed slots (from DB)
    ├─> For each slot:
    │     ├─> beacon_node.is_slot_finalized()
    │     ├─> beacon_node.balances_for_slot()
    │     └─> session.execute(INSERT INTO balance)
    └─> sleep(300)
```

#### 3. Block Rewards Indexer (`src/indexer/block_rewards/main.py`)

**Entry Point**: `python src/indexer/block_rewards/main.py`

**Flow**:
```
while True:
  index_block_rewards()
    ├─> Get all slots from START_SLOT to head_finalized
    ├─> Remove already indexed slots
    ├─> For each slot (parallel processing with semaphore):
    │     ├─> process_slot(slot)
    │     │     ├─> beacon_node.get_slot_proposer_data()
    │     │     ├─> get_block_reward_value()
    │     │     │     ├─> Check for MEV via mev_relay.get_delivered_payload()
    │     │     │     ├─> Calculate balance changes
    │     │     │     ├─> Handle smart contract fee recipients (RP, Lido, etc.)
    │     │     │     └─> Detect MEV bots
    │     │     └─> session.merge(BlockReward)
    └─> sleep(60)
```

#### 4. Validator Indexer (`src/indexer/validators.py`)

**Entry Point**: `python src/indexer/validators.py`

**Flow**:
```
while True:
  index_validators()
    ├─> beacon_node.get_validators()
    └─> session.execute(INSERT INTO validator)
  sleep(3600)
```

#### 5. Withdrawal Indexer (`src/indexer/withdrawals.py`)

**Entry Point**: `python src/indexer/withdrawals.py`

**Flow**:
```
while True:
  index_withdrawals()
    ├─> Get latest slot from DB
    ├─> For each slot from latest to head_finalized:
    │     ├─> beacon_node.withdrawals_for_slot()
    │     └─> session.execute(INSERT INTO withdrawal)
    └─> sleep(60)
```

#### 6. Rocket Pool Indexer (`src/indexer/rocket_pool/main.py`)

**Entry Point**: `python src/indexer/rocket_pool/main.py`

**Flow**:
```
while True:
  run()
    ├─> rocket_pool_data.get_nodes()
    ├─> rocket_pool_data.get_minipools()
    ├─> rocket_pool_data.get_bond_reductions()
    └─> rocket_pool_data.get_reward_snapshots()
  sleep(300)
```

#### 7. Price Indexer (`src/indexer/prices.py`)

**Entry Point**: `python src/indexer/prices.py`

**Flow**:
```
while True:
  index_prices()
    ├─> coin_gecko.get_historical_prices()
    └─> session.execute(INSERT INTO price)
  sleep(3600)
```

---

## Detailed Function Documentation

### Core Provider Classes

#### BeaconNode (`src/providers/beacon_node.py`)

**Purpose**: Interfaces with the Ethereum Consensus Layer (beacon chain)

**Key Methods**:

- `slot_for_datetime(dt)`: Converts datetime to slot number
  - **Input**: datetime object (must be timezone-aware)
  - **Output**: Integer slot number
  - **Location**: `src/providers/beacon_node.py:71`

- `datetime_for_slot(slot, timezone)`: Converts slot to datetime
  - **Input**: slot number, timezone
  - **Output**: datetime object
  - **Location**: `src/providers/beacon_node.py:86`

- `balances_for_slot(slot, validator_indexes=None)`: Gets validator balances
  - **Input**: slot number, optional list of validator indexes
  - **Output**: List of Balance objects
  - **Location**: Used in `balances.py:124`

- `activation_slots_for_validators(validator_indexes, cache)`: Gets activation slots
  - **Input**: list of validator indexes, Redis cache
  - **Output**: Dict mapping validator_index → activation_slot
  - **Usage**: Determines when validators became active

- `get_slot_proposer_data(slot)`: Gets block proposer information
  - **Output**: SlotProposerData(slot, proposer_index, fee_recipient, block_number, block_hash)
  - **Usage**: Critical for reward calculation

- `withdrawals_for_slot(slot)`: Gets all withdrawals in a slot
  - **Output**: List of Withdrawal objects
  - **Location**: Used in `withdrawals.py:48`

#### ExecutionNode (`src/providers/execution_node.py`)

**Purpose**: Interfaces with the Ethereum Execution Layer (EL)

**Key Methods**:

- `get_block(block_number, verbose=False)`: Fetches block data
  - **Input**: block number, verbosity flag
  - **Output**: Block data including transactions
  - **Usage**: For MEV detection and reward calculation

- `get_balance(address, block_number, use_infura=False)`: Gets ETH balance
  - **Input**: Ethereum address, block number
  - **Output**: Balance in wei
  - **Usage**: Calculate balance changes for MEV/rewards

- `eth_call(params, use_infura=True)`: Makes contract calls
  - **Input**: RPC call parameters
  - **Output**: Contract call result
  - **Usage**: Query smart contract state (e.g., Rocket Pool contracts)

- `get_block_receipts(block_number)`: Gets transaction receipts
  - **Input**: block number
  - **Output**: List of transaction receipts
  - **Usage**: Analyze transaction outcomes

#### DbProvider (`src/providers/db_provider.py`)

**Purpose**: Centralized database query interface

**Key Methods**:

- `balances(slots, validator_indexes)`: Query balances
- `block_rewards(min_slot, max_slot, proposer_indexes)`: Query block rewards
- `withdrawals(min_slot, max_slot, validator_indexes)`: Query withdrawals
- `minipools_for_validators(validator_indexes)`: Get Rocket Pool minipool data
- `rocket_pool_node_rewards_for_minipools(...)`: Get RP reward data

#### RocketPoolDataProvider (`src/providers/rocket_pool.py`)

**Purpose**: Interfaces with Rocket Pool smart contracts

**Key Methods**:

- `get_nodes(known_node_addresses, block_number)`: Get all RP nodes
  - **Returns**: List of (node_address, fee_distributor)

- `get_minipools(...)`: Get minipool creation events
  - **Returns**: Dict of node_address → list of minipools

- `get_bond_reductions(from_block, to_block)`: Get bond reduction events
  - **Returns**: List of bond reduction data

- `get_reward_snapshots(start_at_period)`: Get RP reward periods
  - **Returns**: Reward tree data from IPFS

- `get_minipool_node_fee(minipool_address, block_number)`: Get fee percentage
- `get_minipool_bond(minipool_address, block_number)`: Get bond amount

### Reward Calculation Functions

#### Consensus Layer Rewards (`src/api/api_v2/endpoints/rewards.py:420`)

**Function**: `rewards()` - Full endpoint

**Logic**:
1. Get initial balance (at start date or activation)
2. Get end-of-day balances for each day
3. Calculate reward: `(end_balance - start_balance) + withdrawals`
4. Withdrawals are added back because they reduce the balance
5. Full withdrawals (>8 ETH): Only count amount above 32 ETH as reward

**Formula**:
```
daily_consensus_reward = (EOD_balance - previous_balance) + withdrawals_during_day
```

#### Execution Layer Rewards (`src/api/api_v2/endpoints/rewards.py:565`)

**Logic**:
1. Get all block proposals by validator
2. For each block:
   - If MEV detected: reward = `mev_reward_value_wei`
   - If no MEV: reward = `priority_fees_wei`
3. Sum by date

**MEV Detection** (`src/indexer/block_rewards/block_rewards_mev_simple.py`):
1. Check MEV relay for delivered payload
2. If found: Use relay data for MEV recipient and value
3. If not found: Analyze balance changes and transaction patterns
4. Handle special cases: Rocket Pool, Lido, Kraken fee distributors

#### Rocket Pool Rewards (`src/api/api_v2/endpoints/rewards.py:268`)

**Function**: `rewards()` - Rocket Pool endpoint

**Special Handling**:
1. **Withdrawals**: Node operator only gets their share based on bond/fee ratio
2. **Formula**: `node_share = withdrawal * (bond/32 + (32-bond)/32 * fee)`
3. **Block Proposals**:
   - Smoothing pool proposals: Skip (accounted in 28-day rewards)
   - Fee distributor proposals: Calculate node share via balance delta
4. **28-day Rewards**: From Rocket Pool reward trees (RPL + smoothing pool ETH)

---

## Taxation Logic Overview

### Layman's Overview

**The Problem**: When you stake Ethereum, you earn rewards. Tax authorities want to know how much you earned so they can tax it as income.

**The Challenge**: Ethereum staking rewards are complicated:
- You earn rewards every ~6 minutes (every epoch)
- Rewards automatically compound (get added to your stake)
- You can't access rewards until you withdraw
- There are multiple types of rewards (consensus layer, execution layer, MEV)
- Special cases exist (Rocket Pool, validator exits, penalties)

**What This Tool Does**:
1. **Tracks your validator balance** every day at midnight
2. **Calculates daily income** by comparing yesterday's balance to today's balance
3. **Accounts for withdrawals** (money you took out)
4. **Tracks block proposals** (extra rewards when you propose a block)
5. **Detects MEV** (extra profit from transaction ordering)
6. **Converts to fiat** (USD, EUR, etc.) using historical prices
7. **Generates reports** for your tax filing

### Key Tax Concepts

#### 1. **Accrual Basis Income**

Most tax jurisdictions treat staking rewards as income when they are **earned**, not when withdrawn.

- Every day your validator earns ~0.0001-0.0002 ETH
- This is taxable income on the day it's earned
- Value is calculated using the ETH price on that day

#### 2. **Consensus Layer Rewards**

These are the base staking rewards from validating the beacon chain:
- Attestation rewards (most frequent)
- Block proposal rewards (consensus layer portion)
- Sync committee rewards (if selected)

**Calculation**: End-of-day balance minus previous balance, plus any withdrawals

#### 3. **Execution Layer Rewards**

These are rewards from transaction fees:
- Priority fees (tips) from transactions
- MEV (Maximum Extractable Value) from transaction ordering

**Calculation**: Sum of priority fees and MEV when your validator proposes a block

#### 4. **Withdrawals**

Post-Shanghai (April 2023), validators can withdraw:
- **Partial withdrawals**: Automatic, skim rewards above 32 ETH
- **Full withdrawals**: Exit validator, get all 32+ ETH back

**Tax Treatment**:
- Rewards portion: Already taxed when earned (not taxed again)
- Principal (original 32 ETH): Not taxed on withdrawal
- Need to track basis for capital gains when you sell

#### 5. **Rocket Pool Specifics**

Rocket Pool node operators:
- Only deposit 8-16 ETH (protocol deposits the rest)
- Share rewards with protocol based on bond size and commission
- Get additional RPL token rewards
- Have more complex calculations

**Calculation**: This tool calculates only the node operator's share of rewards, not the full minipool rewards.

### How the Tool Calculates Income

#### Step 1: Daily Balance Tracking

```
Day 1: 32.0000 ETH
Day 2: 32.0001 ETH  → Earned 0.0001 ETH
Day 3: 32.0002 ETH  → Earned 0.0001 ETH
Day 4: 32.1000 ETH, proposed block → Earned 0.0998 ETH
```

#### Step 2: Account for Withdrawals

```
Day 100: Balance: 32.2000 ETH, Withdrawal: 0.2000 ETH
  Actual balance: 32.0000 ETH
  Income for day: (32.0000 - 32.0001) + 0.2000 = 0.1999 ETH
```

#### Step 3: Add Execution Layer Rewards

```
Day 4: Proposed block, priority fees = 0.05 ETH
  Total income: 0.0998 (CL) + 0.05 (EL) = 0.1498 ETH
```

#### Step 4: Convert to Fiat

```
Day 4 ETH price: $2,000
Income in USD: 0.1498 ETH × $2,000 = $299.60
```

### Edge Cases Handled

1. **Penalties/Slashing**: Negative income days (balance decreases)
2. **Validator Exits**: Distinguishes between rewards and principal return
3. **MEV**: Properly attributes MEV rewards to the validator
4. **Smart Contract Recipients**: Handles Rocket Pool, Lido, Kraken fee distributors
5. **Missed Blocks**: Handles empty slots (no block proposed)
6. **Bond Reductions**: Rocket Pool bond reductions change reward split

---

## GitHub Issues Analysis

### Issue Categories and Surface Area

Based on analysis of 12 open issues:

#### 1. **Reward Calculation Errors** (High Impact, Complex)
- **#116**: "500 execution layer rewards not available"
  - **Surface Area**: Affects block reward indexer when EL data unavailable
  - **Impact**: Blocks tax report generation
  - **Root Cause**: Missing or failed execution node queries

- **#113**: "Partial withdrawal causing 500 error"
  - **Surface Area**: Withdrawal processing logic
  - **Impact**: Report fails for validators with specific withdrawal patterns
  - **Root Cause**: Edge case in withdrawal classification

- **#94**: MEV recipient validation failure
  - **Surface Area**: MEV detection logic in `block_rewards_mev_simple.py`
  - **Impact**: False rejection of valid MEV addresses
  - **Root Cause**: Incomplete MEV recipient whitelist

#### 2. **Feature Completeness** (Medium Impact, Moderate Complexity)
- **#100**: Rocket Pool mode broken
  - **Surface Area**: Entire Rocket Pool endpoint (`/api/v2/rewards/rocket_pool`)
  - **Impact**: Rocket Pool users cannot get accurate reports
  - **Root Cause**: Changes to RP contracts or calculation logic

- **#115**: EIP-7251 validator consolidation support
  - **Surface Area**: Requires new indexing logic and reward calculation
  - **Impact**: Future validators using consolidation will have incorrect income
  - **Root Cause**: New Ethereum feature not yet supported

#### 3. **Data Classification Issues** (Medium Impact, Low-Medium Complexity)
- **#105**: Compounding mode misclassifies deposits as rewards
  - **Surface Area**: Withdrawal classification logic
  - **Impact**: Overstates income at validator exit
  - **Root Cause**: Doesn't distinguish user deposits from rewards

- **#102**: Large withdrawals marked as exits
  - **Surface Area**: Withdrawal classification (8 ETH threshold)
  - **Impact**: Incorrect income calculation for large partial withdrawals
  - **Root Cause**: Simple threshold doesn't account for all scenarios

#### 4. **UX/Export Issues** (Low Impact, Low Complexity)
- **#74**: Missing fiat values in CSV
  - **Surface Area**: CSV export function
  - **Impact**: Users have to manually add prices
  - **Root Cause**: Export logic doesn't include price data

- **#63**: Static date defaults
  - **Surface Area**: Frontend date picker
  - **Impact**: Poor UX (always defaults to 2023)
  - **Root Cause**: Hardcoded default dates

- **#62**: No URL query parameters
  - **Surface Area**: Frontend routing
  - **Impact**: Can't bookmark/share specific queries
  - **Root Cause**: State not reflected in URL

- **#15**: Limited export formats
  - **Surface Area**: Export functionality
  - **Impact**: Users need specific formats for tax software
  - **Root Cause**: Only one format currently supported

#### 5. **Data Quality** (Low Impact, Low Complexity)
- **#22**: Decimal precision
  - **Surface Area**: Database schema, calculations
  - **Impact**: Potential rounding errors
  - **Root Cause**: Float vs Decimal handling

### Surface Area Summary

**Most Complex Areas** (most issues):
1. **Reward calculation logic** - Core tax calculation
2. **Withdrawal classification** - Distinguishing reward types
3. **MEV detection** - Complex heuristics
4. **Rocket Pool integration** - Additional protocol complexity

**Most Stable Areas** (few issues):
1. **Balance indexing** - Simple, reliable
2. **Price fetching** - Straightforward API calls
3. **Validator indexing** - Direct beacon chain queries

---

## Adding Network Upgrade Support

### Example: Adding Support for a New Network Upgrade

#### Scenario 1: New Withdrawal Type (e.g., EIP-7251 Consolidation)

**What needs to change**:

1. **Database Schema** (`src/db/tables.py`)
   ```python
   # Add new column to Withdrawal table
   withdrawal_type = Column(String(20), nullable=True)  # 'partial', 'full', 'consolidation'
   ```

2. **Migration**
   ```bash
   make migration-generate MIGRATION_NAME="add withdrawal type"
   make migrate
   ```

3. **Withdrawal Indexer** (`src/indexer/withdrawals.py`)
   - Update `withdrawals_for_slot()` to detect new withdrawal type
   - Add logic to classify consolidations

4. **Reward Calculation** (`src/api/api_v2/endpoints/rewards.py`)
   - Update withdrawal processing logic
   - Handle consolidation withdrawals differently
   - Ensure they're not counted as income

5. **Testing**
   - Add test cases for new withdrawal type
   - Verify income calculations are correct

**Files to modify**:
- `src/db/tables.py`
- `src/indexer/withdrawals.py`
- `src/api/api_v2/endpoints/rewards.py`
- `tests/api/api_v2/endpoints/test_rewards_v2.py`

#### Scenario 2: New Validator Type (e.g., Pectra 0x02 Validators)

**What needs to change**:

1. **Validator Metadata** (`src/db/tables.py`)
   ```python
   class Validator(Base):
       # Add new fields
       validator_type = Column(String(10))  # '0x01', '0x02'
       max_effective_balance = Column(Integer)  # 32 ETH or 2048 ETH
   ```

2. **Validator Indexer** (`src/indexer/validators.py`)
   - Parse new validator credential types from beacon chain
   - Store validator type

3. **Balance Indexer** (`src/indexer/balances.py`)
   - Handle different max effective balances
   - Adjust end-of-day balance calculations if needed

4. **Reward Calculation** (`src/api/api_v2/endpoints/rewards.py`)
   - Account for different balance limits
   - Adjust withdrawal classification (e.g., full withdrawal threshold)

**Files to modify**:
- `src/db/tables.py`
- `src/indexer/validators.py`
- `src/indexer/balances.py`
- `src/api/api_v2/endpoints/rewards.py`

#### Scenario 3: Adding Rocket Pool Feature (e.g., New Bond Amount)

**What needs to change**:

1. **Contract Addresses** (`src/providers/rocket_pool.py`)
   ```python
   _MINIPOOL_MANAGER_ADDRESSES = [
       # ... existing
       {
           # v6 deployed on YYYY-MM-DD
           "address": "0x...",
       },
   ]
   ```

2. **Bond Calculation Logic** (`src/api/api_v2/endpoints/rewards.py`)
   - Update `_get_rocket_pool_reward_share_withdrawal_for_bond_fee()`
   - Handle new bond amounts (e.g., 4 ETH, 1.5 ETH)

3. **Testing**
   - Test with new bond amounts
   - Verify fee sharing calculations

**Files to modify**:
- `src/providers/rocket_pool.py`
- `src/api/api_v2/endpoints/rewards.py`
- `tests/provider/test_rocket_pool.py`

### General Process for Network Upgrades

1. **Research the Change**
   - Read the EIP
   - Understand consensus layer changes
   - Understand execution layer changes
   - Identify impact on tax calculations

2. **Identify Affected Components**
   - Which indexers need updates?
   - Does the database schema change?
   - Does reward calculation logic change?

3. **Update Data Models**
   - Modify `src/db/tables.py`
   - Generate migration
   - Test migration on dev database

4. **Update Indexers**
   - Modify relevant indexers
   - Add new data collection if needed
   - Handle backward compatibility

5. **Update Business Logic**
   - Modify reward calculation
   - Update API endpoints
   - Handle edge cases

6. **Test Thoroughly**
   - Unit tests
   - Integration tests
   - Manual testing with real data

7. **Deploy**
   - Migrate database
   - Deploy new code
   - Monitor for errors

---

## CL+EL Stack Details

### Current Stack Configuration

From `docker-compose.yml:154-206`:

#### Consensus Layer (CL) - Lighthouse

```yaml
beacon_node:
  image: sigp/lighthouse:v7.0.1
  command:
    - lighthouse
    - --network mainnet
    - bn
    - --http
    - --http-address 0.0.0.0
    - --metrics
    - --execution-endpoint=http://geth:8551
    - --execution-jwt=/tmp/jwt_secret
    - --checkpoint-sync-url=https://mainnet.checkpoint.sigp.io
    - --reconstruct-historic-states      # Critical for archive mode
    - --disable-backfill-rate-limiting
```

**Key Flags**:
- `--reconstruct-historic-states`: Enables reconstruction of historical state for API queries
- `--checkpoint-sync-url`: Fast sync from checkpoint
- `--execution-endpoint`: Connects to Geth via authenticated RPC

**Data Requirements**:
- ethstaker.tax requires "archive-level" data for **recent epochs** (few days)
- Full archive mode back to genesis takes ~670GB (Lighthouse)
- Only needs last 2-10 weeks of archive data

#### Execution Layer (EL) - Geth

```yaml
geth:
  image: ethereum/client-go:v1.15.11
  command:
    - --datadir=/data
    - --http
    - --http.addr=0.0.0.0
    - --authrpc.addr=geth
    - --authrpc.jwtsecret=/tmp/jwt_secret
    - --metrics
```

**Notes**:
- Standard Geth configuration
- Could be replaced with any EL client supporting history expiry
- JWT authentication for CL-EL communication

### Why Not eth-docker?

From your quote:
> "I'm personally not a big fan of eth-docker due to the things it does in the background *automagically*. I prefer to know exactly which flags are being set for the clients at all times."

**Reasoning**:
- **Transparency**: Direct docker-compose.yml shows exact configuration
- **Control**: Can modify any flag without wrapper abstractions
- **Debugging**: Easier to troubleshoot when you see exact commands
- **Customization**: Can deviate from standard configs easily

### Future CL Client Requirements

**Ideal CL Client Features**:
1. **Limited Archive Mode**: Prune archive data older than 2-10 weeks
2. **Space Efficient**: <2TB total for full stack
3. **Stable**: Reliable uptime, no data corruption
4. **Archive API**: Support historical state queries (not just finalized head)

**Current State (2024)**:
- **Lighthouse**: Archive mode = full history to genesis (~670GB)
- **Prysm**: Similar archive requirements
- **Teku**: Similar archive requirements
- **Nimbus**: Lighter weight but needs verification of archive capabilities
- **Caplin** (Erigon's CL): Unknown if it has archive mode at all

**Challenge**: Most CL clients don't support "sliding window archive mode" (keeping only recent history).

### External RPC Providers

**Why Use External EL RPC**:
- Can use history expiry for local EL node (saves space)
- Offload archive queries to Infura/Alchemy
- Reduces local storage requirements

**Current Code Support**:
```python
# src/providers/execution_node.py
if os.getenv("EXECUTION_NODE_USE_INFURA_EVERYWHERE") == "true":
    return os.getenv("EXECUTION_NODE_INFURA_ARCHIVE_URL")
```

**Trade-offs**:
- **Pro**: Much less storage needed locally
- **Con**: Dependent on third-party availability
- **Con**: Rate limits on external providers
- **Con**: Cost for high-volume queries

### Archive Data Requirements

**What ethstaker.tax Needs**:

1. **Beacon Chain Data** (last few days):
   - Validator balances at arbitrary historical slots
   - Block proposals and attestations
   - Withdrawal events
   - All available via beacon node API

2. **Execution Chain Data** (finalized blocks):
   - Transaction receipts
   - Contract state at specific blocks
   - Balance queries at historical blocks
   - Can be outsourced to Infura/Alchemy

**Why Recent History Matters**:
- Downtime recovery: If indexer is down for a few days
- Reindexing: May need to reprocess recent data
- Validation: Double-check calculations

**Recommendation**:
- Keep 2-4 weeks of full CL archive data
- Use checkpoint sync + reconstruct recent states
- Offload EL archive queries to external provider

---

## API Documentation

### API v2 Endpoints

Base URL: `/api/v2`

#### POST `/api/v2/rewards/full`

**Purpose**: Get complete validator rewards (solo stakers)

**Request Body**:
```json
{
  "validator_indexes": [123, 456, 789],
  "start_date": "2023-01-01",
  "end_date": "2023-12-31",
  "expected_fee_recipient_addresses": ["0x..."]  // Optional
}
```

**Response**:
```json
{
  "validator_rewards_list": [
    {
      "validator_index": 123,
      "consensus_layer_rewards": [
        {
          "date": "2023-01-01",
          "amount_wei": "100000000000000000"  // 0.1 ETH
        }
      ],
      "execution_layer_rewards": [
        {
          "date": "2023-05-15",
          "amount_wei": "50000000000000000"  // 0.05 ETH
        }
      ],
      "withdrawals": [
        {
          "date": "2023-06-01",
          "amount_wei": "200000000000000000"  // 0.2 ETH
        }
      ]
    }
  ]
}
```

**Implementation**: `src/api/api_v2/endpoints/rewards.py:409`

**Logic**:
1. Validate date range
2. Get activation slots for all validators
3. Get initial balances
4. Get end-of-day balances for date range
5. Get all withdrawals in range
6. Get all block rewards in range
7. For each validator, for each day:
   - Calculate consensus reward: `(end_balance - start_balance) + withdrawals`
   - Add execution layer rewards from block proposals
8. Return structured data

**Rate Limit**: 100 requests per hour

#### POST `/api/v2/rewards/rocket_pool`

**Purpose**: Get Rocket Pool node operator rewards

**Request Body**:
```json
{
  "validator_indexes": [123, 456],
  "start_date": "2023-01-01",
  "end_date": "2023-12-31"
}
```

**Response**:
```json
{
  "validator_rewards_list": [
    {
      "validator_index": 123,
      "execution_layer_rewards": [...],
      "withdrawals": [...]
    }
  ],
  "rocket_pool_node_rewards": [
    {
      "date": "2023-01-15T00:00:00Z",
      "node_address": "0x...",
      "amount_wei": "1000000000000000000",  // 1 ETH smoothing pool
      "amount_rpl": "100000000000000000000"  // 100 RPL
    }
  ]
}
```

**Implementation**: `src/api/api_v2/endpoints/rewards.py:264`

**Special Logic**:
1. Get Rocket Pool minipool data for validators
2. For withdrawals:
   - Calculate node operator share based on bond/fee at time of withdrawal
   - Formula: `node_share = amount × (bond/32 + (32-bond)/32 × fee)`
3. For block proposals:
   - Skip if went to smoothing pool
   - Calculate share from fee distributor balance delta
4. Add 28-day Rocket Pool rewards (RPL + smoothing pool ETH)

**Rate Limit**: 100 requests per hour

#### GET `/api/v2/prices`

**Purpose**: Get historical ETH prices

**Query Parameters**:
- `token`: Token symbol (e.g., "eth")
- `currency`: Fiat currency (e.g., "usd", "eur")
- `from_timestamp`: Unix timestamp
- `to_timestamp`: Unix timestamp

**Response**:
```json
{
  "prices": [
    {
      "timestamp": "2023-01-01T00:00:00Z",
      "value": "1200.50"
    }
  ]
}
```

**Implementation**: `src/api/api_v2/endpoints/prices.py`

### Key API Design Patterns

1. **POST for large requests**: Validator lists can be long, exceeds URL length limits
2. **Wei denomination**: All ETH amounts in wei to avoid decimals
3. **Date-based aggregation**: Rewards aggregated by day for tax reporting
4. **Caching**: Heavy use of Redis caching for repeated queries
5. **Rate limiting**: Prevents abuse, 100 req/hour for compute-heavy endpoints
6. **Error handling**: Returns 500 with details when data unavailable

### Database Schema

See `src/db/tables.py` for full schema.

**Key Tables**:

- **balance**: `(slot, validator_index) → balance`
- **block_reward**: `(slot) → proposer_index, fee_recipient, priority_fees_wei, mev, mev_reward_*`
- **withdrawal**: `(id) → slot, validator_index, amount_gwei, withdrawal_address_id`
- **validator**: `(validator_index) → pubkey`
- **rocket_pool_minipool**: `(minipool_address) → validator_pubkey, bond, fee, node_address`
- **rocket_pool_bond_reduction**: `(minipool_address, new_bond_amount) → timestamp, new_fee`
- **rocket_pool_reward**: `(node_address, reward_period_index) → collateral_rpl, smoothing_pool_wei`
- **price**: `(token, currency, timestamp) → value`

**Indexes**:
- `balance_pkey`: (slot, validator_index) - primary key
- `validator.pubkey`: For lookup by public key
- `withdrawal.validator_index`: For filtering withdrawals by validator

---

## Monitoring and Debugging

### Prometheus Metrics

Exposed on `/metrics` endpoint:

**Balance Indexer**:
- `slots_with_missing_balances`: Remaining slots to index

**Block Reward Indexer**:
- `slots_with_missing_block_rewards`: Remaining slots to index
- `slots_indexing_failures`: Failed slots
- `slot_being_indexed`: Current slot

**Rocket Pool Indexer**:
- `rocket_pool_last_reward_period_indexed`: Last indexed period
- `rocket_pool_nodes`: Node count
- `rocket_pool_minipools`: Minipool count
- `rocket_pool_bond_reductions`: Bond reduction count

**API**:
- HTTP request duration histogram
- Request count by endpoint
- `beaconchain_request_count`: Requests to beaconcha.in
- `beacon_node_request_count`: Requests to beacon node
- `exec_node_request_count`: Requests to execution node

### Grafana Dashboards

Available at http://localhost:3000

Preconfigured dashboards show:
- Indexer progress
- API request rates
- Database performance
- Node sync status

### Logging

Configured in `etc/logging.yml`

**Log Levels**:
- INFO: Normal operations, progress updates
- WARNING: Recoverable issues (e.g., MEV relay down)
- ERROR: Failed operations (e.g., unable to process slot)
- DEBUG: Detailed trace (not enabled by default)

**Key Log Locations**:
- stdout/stderr in Docker containers
- Can be collected by Docker logging drivers

---

## Development Workflow

### Making Schema Changes

1. Edit `src/db/tables.py`
2. Generate migration: `make migration-generate MIGRATION_NAME="description"`
3. Review migration in `alembic/versions/`
4. Apply migration: `make migrate`

### Dependency Changes

1. Edit `requirements.in`
2. Compile: `make compile-dependencies`
3. Or upgrade: `make upgrade-dependencies PACKAGE_NAME=httpx`

### Running Tests

```bash
# All tests
pytest

# Specific test file
pytest tests/api/api_v2/endpoints/test_rewards_v2.py

# With coverage
pytest --cov=src
```

### Local Development

```bash
# Start all services
docker-compose up

# Start only specific services
docker-compose up api db redis

# Rebuild after code changes
docker-compose up --build

# View logs
docker-compose logs -f api
docker-compose logs -f indexer_balances
```

---

## Summary

**ethstaker.tax** is a production-grade tax calculation tool for Ethereum stakers. It combines:
- **Robust indexing**: Continuously indexes all relevant blockchain data
- **Accurate calculations**: Handles complex reward scenarios including MEV and Rocket Pool
- **Tax compliance**: Generates reports suitable for tax filing
- **Scalability**: Handles thousands of validators
- **Reliability**: Uses finalized data only, extensive error handling
- **Monitoring**: Full observability with Prometheus/Grafana

The codebase is well-structured with clear separation between indexing, data access, and API layers. The main complexity lies in the reward calculation logic, particularly for edge cases and protocol-specific features like Rocket Pool.
