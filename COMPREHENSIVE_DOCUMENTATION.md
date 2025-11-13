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

*[This section will be populated with detailed function descriptions in the next commit]*

---

## Taxation Logic Overview

*[This section will be populated with taxation concepts in the next commit]*

---

## GitHub Issues Analysis

*[This section will be populated with issue analysis in the next commit]*

---

## Adding Network Upgrade Support

*[This section will be populated with upgrade instructions in the next commit]*

---

## CL+EL Stack Requirements

*[This section will be populated with stack details in the next commit]*

---

## API Code Deep Dive

*[This section will be populated with API analysis in the next commit]*

---

*Documentation in progress - this is a living document that will be updated as the codebase is analyzed further.*
