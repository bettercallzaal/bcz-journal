---
topic: inspiration
type: comparison
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647f - Financial / Governance Cluster

## TL;DR

Mental Wealth Academy maintains 3 independent but architecturally related financial/governance repos created between 2026-02-13 and 2026-02-27. **treasury** (55 KB TypeScript + 27 KB CSS) is a live prediction-market trading engine on mwa-treasury.vercel.app using Black-Scholes binary pricing + Kelly criterion + Avellaneda-Stoikov market-making with real-time Polymarket CLOB integration. **academy-crm** (3.8 MB Solidity, 1.6 MB JavaScript) is the Azura autonomous treasury agent on Chainlink CRE, managing multi-asset reserves across 6 crypto/RWA pairs and writing consensus-signed reports on-chain via CRE WASM execution. **moltbook-research** (52 KB TypeScript) is the 2026 Chainlink Convergence Hackathon implementation (CRE × AI track) that ingests data from 4+ real-world APIs (Meta Transparency Center, PubMed, news feeds, DAO treasury), analyzes harm through neuroscience + systemic inequality frameworks, and generates 5th-grade-level governance proposals. 

Treasury predates academy-crm by 14 days and moltbook by 3 days. BlueMarketTrader.sol in the platform repo is a simpler on-chain treasury executor (receive USDC, trade binary outcomes via KeystoneForwarder) - no pricing models, no market data integration, no Python/ML stack. treasury repo owns the pricing + data logic; academy-crm owns the CRE orchestration + RWA bridge; moltbook owns the DAO research + participatory governance layer.

## Repo Relationships

```
Created Timeline:

academy-crm (Solidity primary)
  └── 2026-02-13 01:15:32Z

treasury (TypeScript primary)
  └── 2026-02-27 21:07:11Z
       (Deployed live 2026-02-27 22:19:31Z)

moltbook-research (TypeScript primary)
  └── 2026-02-24 10:31:41Z

Platform (Governance + Game)
  ├── contracts/BlueMarketTrader.sol (executor only)
  ├── contracts/BlueKillStreak.sol (game scoring)
  └── contracts/EtherealHorizonPathway.sol (onboarding)


Dependency Flow:

academy-crm (CRE orchestrator)
  ├── Treasury Workflow (reads multi-asset, writes AzuraTreasury.sol)
  ├── Azura-Workflow (legacy, archived)
  └── contracts/
       ├── AzuraToken.sol (0x31E56...) ERC20 mint/burn
       ├── AzuraTreasury.sol (0x4DdE...) Report storage + backing ratio
       └── AzuraTreasuryProxy.sol (0xac32...) CRE report receiver

treasury (Standalone execution engine)
  ├── lib/trading-engine.ts (Black-Scholes, Kelly, edge detection)
  ├── lib/polymarket-clob.ts (HMAC-SHA256 signed CLOB orders)
  ├── lib/market-api.ts (CoinGecko + Polymarket aggregation)
  └── Live deployment: mwa-treasury.vercel.app

moltbook-research (Governance layer, hackathon)
  ├── research-governance-workflow/
  │   ├── main.ts (CronTrigger every 6h)
  │   ├── research.ts (4-API ingestion with consensus)
  │   ├── analysis.ts (Harm signal detection)
  │   ├── proposals.ts (Plain-language proposal generator)
  │   └── writer.ts (On-chain CRE execution)
  └── Not yet deployed to mainnet/testnet (hackathon-only)


Platform integration points:

BlueMarketTrader.sol (platform/contracts/src/)
  └── Simpler pattern: receive USDC → execute trade via KeystoneForwarder
      (no pricing models, no market data, no Kelly criterion)
      Differences from treasury: no edge detection, no Polymarket integration,
      no Black-Scholes, no position sizing
```

## treasury (Standalone Next.js App)

**Repo:** https://github.com/Mental-Wealth-Academy/treasury  
**Live:** https://mwa-treasury.vercel.app  
**Created:** 2026-02-27 21:07:11Z  
**Last Push:** 2026-02-27 22:19:31Z  
**Language:** TypeScript (95,157 bytes) + CSS (27,270 bytes)  
**Primary:** Next.js 14 (App Router)

### Purpose
Autonomous prediction-market trading engine for binary-outcome crypto markets. Prices markets using Black-Scholes binary model, sizes positions via Kelly criterion with half-Kelly drawdown control, and quotes two-sided liquidity through Avellaneda-Stoikov optimal spread. Only executes trades when model fair value diverges by 3%+ from market price.

### Architecture

```
app/
  api/treasury/          9 API routes for market data, signals, execution, P&L
  treasury/page.tsx      Main dashboard UI (portfolio table, position cards, charts)

components/
  treasury-display/      Portfolio table + position cards
  treasury-how-to/       Onboarding walkthrough overlay
  soul-gems/             Gem-based reward visualisations

lib/
  market-api.ts          CoinGecko + market-data aggregation
  trading-engine.ts      Black-Scholes pricer, Kelly sizer, AS quoter
  polymarket-clob.ts     Polymarket CLOB REST client (HMAC-SHA256 auth)
  apple-holders.ts       Holder-snapshot utilities
  execution-log-store.ts Trade execution journal
```

### Trading Pipeline (5 Steps)

1. **Fetch** — Pull live order-book + trades from Polymarket CLOB; spot prices from CoinGecko
2. **Price** — Run Black-Scholes binary pricer: fair value = S*N(d2) where S = spot, d2 = delta, N(·) = normal CDF
3. **Size** — Kelly criterion: position_size = (edge / odds) * Kelly_fraction (capped at half-Kelly = 25%)
4. **Quote** — Avellaneda-Stoikov sets optimal bid/ask spread around fair value
5. **Execute** — If edge > 3%, place limit order on Polymarket CLOB; log to execution store

### Key Constants (from trading-engine.ts)

- **SIGMA** = 0.50 (volatility surface constant)
- **T_EXP** = 0.0000095 (time-to-expiration exponent)
- **R_FREE** = 0.0433 (risk-free rate)
- **GAMMA** = 0.10 (gamma decay constant)
- **SIGMA_B** = 0.328 (bid-ask volatility)
- **K_DECAY** = 1.50 (Kelly decay factor)
- **EDGE_THRESHOLD** = 3.0% (minimum edge for trade execution)
- **KELLY_FRACTION** = 0.25 (half-Kelly applied)
- **MAX_POSITION_PCT** = 5% of trading balance per position
- **MAX_TOTAL_EXPOSURE_PCT** = 40% of balance
- **STOP_LOSS_PCT** = 15%
- **MAX_DRAWDOWN_PCT** = 20%

### Polymarket CLOB Integration

**File:** lib/polymarket-clob.ts  
**Auth:** HMAC-SHA256 using API key + secret + passphrase  
**Endpoints:** Base URL `https://clob.polymarket.com`

Order types:
```typescript
interface ClobOrder {
  tokenID: string;        // Market outcome token
  price: number;          // [0, 1] decimal
  size: number;           // Share quantity
  side: 'BUY' | 'SELL';
  expiration?: number;    // Unix timestamp
}
```

Holder snapshot utilities (apple-holders.ts) track large position holders for signal extraction.

### Dashboard Features

- Portfolio table with position PnL
- Position cards with entry price, size, time-to-expiration
- P&L charts (cumulative, by-market)
- Gem-based reward visualisations (soul-gems component)
- Onboarding walkthrough (treasury-how-to)

### Stack Details

| Layer | Tech |
|-------|------|
| Framework | Next.js 14 (App Router) |
| Language | TypeScript 5 |
| Blockchain | ethers.js v5 |
| Market data | CoinGecko API (live crypto prices) |
| Exchange | Polymarket CLOB (REST + HMAC auth) |
| Styling | CSS Modules + MWA design tokens |

---

## academy-crm (Solidity + CRE Workflows)

**Repo:** https://github.com/Mental-Wealth-Academy/academy-crm  
**Deployed Testnet:** Sepolia (contracts) + CRE testnet (workflows)  
**Created:** 2026-02-13 01:15:32Z  
**Last Push:** 2026-03-22 12:09:33Z  
**Languages:** Solidity (3.8 MB, primary) + JavaScript (1.6 MB) + TypeScript (170 KB) + Python (44 KB)  
**Homepage:** https://azura-theta.vercel.app

### Purpose

Autonomous treasury agent ("Azura") built on Chainlink Runtime Environment (CRE). Reads on-chain balances (WBTC, WETH, native ETH, RWA metals), fetches real-world prices from APIs, computes unified backing ratio with 18-decimal precision, and writes cryptographically verified reports to smart contracts via CRE consensus.

### Core Concept

Azura is a **decentralized oracle program** that:
1. Runs on Chainlink's Decentralized Oracle Network (DON)
2. Compiles to WASM (not EVM)
3. Executes consensus-signed workflows off-chain
4. Settles results on-chain via CRE Report Receiver pattern
5. Updates treasury contracts atomically

### Architecture

```
azura/
├── src/                    CLI source (TypeScript)
│   ├── entry.ts            CLI entry point
│   ├── cli/                Banner, theme, version
│   ├── terminal/           Terminal rendering
│   └── ascii.ts            ASCII art
│
├── azura/
│   ├── treasury-workflow/  Primary CRE workflow (compiles to WASM)
│   │   ├── main.ts         Cron handler + config schema
│   │   ├── fetchers.ts     CoinGecko (crypto) + metals.dev (RWA) API calls
│   │   ├── readers.ts      EVM client balance readers (WBTC, WETH, native, token supply)
│   │   ├── report.ts       Treasury report builder + ABI encoder
│   │   ├── writer.ts       On-chain report writer via CRE consensus
│   │   ├── types.ts        Shared TypeScript types (AssetType enum, configs)
│   │   ├── config.staging.json   Staging network config + contract addresses
│   │   └── workflow.yaml   CRE workflow metadata + trigger schedule
│   │
│   ├── azura-workflow/     Deprecated/archived workflow
│   │
│   └── contracts/          Solidity (Foundry)
│       ├── src/
│       │   ├── AzuraToken.sol         ERC20 with mint/burn (0x31E56...)
│       │   ├── AzuraTreasury.sol      Report storage + state management
│       │   └── AzuraTreasuryProxy.sol CRE report receiver + forwarder
│       ├── script/         Deploy scripts (Deploy.s.sol)
│       ├── test/           Forge tests
│       ├── abi/            TypeScript ABI stubs
│       └── foundry.toml    Foundry config
│
├── web/                    Dashboard & landing page (Next.js 15)
│   └── src/
│       ├── app/            App router pages
│       ├── components/     UI, landing, dashboard components
│       └── lib/            Constants, utilities
│
├── azura.mjs               CLI shebang entry point
├── project.yaml            CRE project config (shared settings)
├── secrets.yaml            Secret-to-env mapping
└── .env                    Private keys & API keys (gitignored)
```

### Treasury Workflow - Data Flow

**Trigger:** CronCapability (schedule from config.staging.json)

**Execution (main.ts):**
```
onCronTrigger(runtime, CronPayload)
  ├── fetchCryptoPrices()     → CoinGecko: BTC, ETH spot prices (USD)
  ├── fetchMetalPrices()      → metals.dev: Gold, Silver, Platinum, Palladium
  ├── readWbtcBalance()       → EVM client: WBTC token balance (wei)
  ├── readWethBalance()       → EVM client: WETH token balance (wei)
  ├── readNativeEthBalance()  → EVM client: native ETH balance (wei)
  ├── readAzuraTotalSupply()  → EVM client: AzuraToken total supply
  ├── buildTreasuryReport()   → Compute USD values + backing ratio
  ├── encodeTreasuryReport()  → ABI encode for Solidity struct
  ├── writeTreasuryReport()   → CRE writer: send signed report on-chain
  └── Return txHash
```

### Supported Assets (6 Total)

| Asset | Type | Price Source | Decimals |
|-------|------|--------------|----------|
| BTC (WBTC) | Crypto | CoinGecko | 8 |
| ETH (WETH + native) | Crypto | CoinGecko | 18 |
| Gold (XAU) | RWA | metals.dev | varies |
| Silver (XAG) | RWA | metals.dev | varies |
| Platinum (XPT) | RWA | metals.dev | varies |
| Palladium (XPD) | RWA | metals.dev | varies |

**Backing Ratio Calculation:** (total_USD_value) / (azura_token_supply) with 18-decimal precision

### Smart Contracts (Sepolia Testnet)

| Contract | Address | Purpose | Lines |
|----------|---------|---------|-------|
| **AzuraToken** | 0x31E56Ac35E34aAD04707e9Bf75E3D7f3d8F5bE44 | ERC20 with mint/burn for treasury backing | ~150 |
| **AzuraTreasury** | 0x4DdE462B7e36Ee1516A2B46c045b83C8d504B951 | Report storage + backing ratio getter | ~100 |
| **AzuraTreasuryProxy** | 0xac32FeFF6aF183d6beBb7426aBcC310DfF36c8D4 | CRE report receiver + forwarder | ~75 |

**AzuraTreasury.sol Interface:**
```solidity
struct Holding {
    uint8 assetType;      // Enum: Crypto vs RWA
    uint256 amount;       // Wei or raw units
    uint256 priceUsd;     // USD price per unit (18 decimals)
    uint256 valueUsd;     // Total USD value of holding
}

struct TreasuryReport {
    Holding[] holdings;
    uint256 totalValueUsd;
    uint256 azuraTotalSupply;
    uint256 backingRatio;  // 18-decimal backing ratio
    uint256 timestamp;
}

function updateReserves(TreasuryReport calldata report) external onlyAuthorized;
function getLatestReport() external view returns (TreasuryReport memory);
function getBackingRatio() external view returns (uint256);
function getTotalValue() external view returns (uint256);
```

### CRE Commands

```bash
# Simulate workflow locally (no on-chain changes)
cre workflow simulate azura/treasury-workflow --target staging-settings --engine-logs

# Deploy workflow to CRE registry (compiles TS → WASM)
cre workflow deploy azura/treasury-workflow --target staging-settings

# Activate on DON (start executing on schedule)
cre workflow activate azura/treasury-workflow --target staging-settings

# Pause running workflow
cre workflow pause azura/treasury-workflow --target staging-settings

# Delete all versions
cre workflow delete azura/treasury-workflow --target staging-settings
```

### Prerequisites & Stack

| Requirement | Version |
|---|---|
| Node.js | >= 22 |
| Bun | >= 1.0 |
| CRE CLI | >= 1.0.10 |
| Foundry | Latest |
| TypeScript | 5.x |

**Dependencies:**
- @chainlink/cre-sdk ^1.0.9
- viem 2.x (Ethereum encoding)
- zod 3.25.76 (Runtime validation)
- ethers.js v5 (legacy balance reads)

### Web Dashboard

Next.js 15 with Tailwind CSS 4 + Framer Motion.
Deployed at: https://azura-theta.vercel.app

Features:
- Cyberpunk data-viz background + Azura character render
- Live treasury stats (total USD value, backing ratio)
- Asset allocation pie chart
- Holdings table with per-asset USD values
- Treasury health indicators

---

## moltbook-research (Hackathon DAO Governance)

**Repo:** https://github.com/Mental-Wealth-Academy/moltbook-research  
**Created:** 2026-02-24 10:31:41Z  
**Last Push:** 2026-02-24 10:31:44Z  
**Languages:** TypeScript (52 KB, primary) + Solidity (6.7 KB) + Shell (512 bytes)  
**Track:** Chainlink Convergence Hackathon (CRE × AI)  
**Deadline:** 2026-03-08 23:59 ET  
**Prize Pool:** 1st $3,500 | 2nd $1,500

### Purpose

Autonomous research + governance agent for Mental Wealth DAO. Ingests data from 4+ real-world APIs, analyzes systemic harm through neuroscience + systemic inequality frameworks, generates evidence-backed governance proposals in 5th-grade-level language, and executes them on-chain via Chainlink CRE.

### Concept: "Beloved Community" Governance

Principles from Grace Lee Boggs framework:
- **Transparent:** Every decision recorded on-chain with full audit trail
- **Participatory:** AI recommends, humans approve (DAO vote, not autocracy)
- **Data-grounded:** Evidence-backed proposals, not opinion
- **Oriented toward flourishing:** Counter extraction with protection

### Architecture

```
research-governance-workflow/
├── main.ts              CronTrigger handler, config schema validation
├── research.ts          Phase 1: Fetch from 4 real APIs with consensus aggregation
├── analysis.ts          Phase 2: Harm signal detection + neuroscience scoring
├── proposals.ts         Phase 3: Governance proposal generator (5th grade level)
├── writer.ts            Phase 4: On-chain execution via CRE
├── types.ts             Shared TypeScript enums + interfaces
├── config.staging.json  Staging network config + API URLs
├── data-sources.md      API documentation + example payloads
├── workflow.yaml        CRE metadata + trigger schedule
└── contracts/           Minimal governance ABI stubs

project.yaml            CRE project config (shared)
secrets.yaml            Secret-to-env mappings
```

### Data Flow

```
CronTrigger (every 6 hours)
    │
    ├─► fetchAllResearch(runtime, timestamp)
    │   ├─► Meta Transparency Center API
    │   │   └─► Enforcement actions, platform harms, violations
    │   ├─► PubMed / NCBI E-utilities
    │   │   └─► Papers: algorithmic harm, adolescent mental health post-2012
    │   ├─► News Feed API
    │   │   └─► Current platform violations, lawsuits, regulatory action
    │   └─► DAO Treasury (on-chain)
    │       └─► Current reserves, available budget
    │
    ├─► analyzeResearch(runtime, research, timestamp)
    │   ├─► Harm Signal Detection
    │   │   ├─► Algorithmic amplification indicators
    │   │   ├─► Body image harm prevalence
    │   │   ├─► Attention extraction metrics
    │   │   ├─► Data exploitation patterns
    │   │   └─► Knowledge suppression evidence
    │   ├─► Neuroscience Impact Assessment
    │   │   ├─► Stress response scoring
    │   │   ├─► Cognitive development impact
    │   │   └─► Executive function risk (prefrontal cortex development 12-25)
    │   ├─► Gender Divergence Detection
    │   │   └─► Disproportionate impact quantification (girls vs boys)
    │   └─► 2012 Inflection Analysis
    │       └─► Deviation from pre-smartphone baseline
    │       └─► Metrics: teen girl suicides +189%, self-harm +151%
    │
    ├─► generateProposal(runtime, analysis, daoTreasury)
    │   ├─► Problem statement (plain language, 5th grade)
    │   ├─► Recommended actions
    │   ├─► Resource allocation suggestions (USD)
    │   ├─► Expected outcomes
    │   └─► Risk analysis
    │
    └─► writeProposal(runtime, proposal)
        ├─► ABI encode proposal struct
        ├─► Create consensus-signed report via CRE
        └─► Write proposal hash on-chain (audit trail)
```

### Data Sources (4 + DAO Treasury)

| Source | API Type | Data | Consensus? |
|--------|----------|------|-----------|
| **Meta Transparency Center** | REST HTTPS | Enforcement actions, takedowns, appeals | Yes (CRE aggregates) |
| **PubMed (NCBI E-utilities)** | REST XML/JSON | Peer-reviewed research queries | Yes |
| **News Feed API** | REST JSON | Real-time articles + regulatory filings | Yes |
| **DAO Treasury (on-chain)** | EVM RPC | Current balances, treasury state | Yes (consensus RPC) |

**Consensus Aggregation:** CRE Decentralized Oracle Network independently fetches each API, compares results, agrees on canonical data before proceeding.

### Harm Frameworks

**2012 Inflection Point Data:**
- Teen girl suicides: +189% post-2012 (pre-2012 baseline = 0)
- Teen self-harm hospitalizations: +151% post-2012
- Boys' rates: +40% post-2012 (slower divergence)
- Platform: Instagram launched 2010, mainstreamed 2012 onwards

**Neuroscience Window:**
- Prefrontal cortex development: ages 12-25
- Social comparison circuits peak: ages 14-18
- Stress hormone (cortisol) sensitivity elevated during adolescence
- Executive function impairment during high-stress periods correlates with long-term planning deficits

**Systemic Inequality Frames (from Grace Lee Boggs):**
- Property tax education funding creates resource scarcity by zip code
- Platforms concentrate wealth while extracting user attention/data
- Girls disproportionately targeted by algorithmic body-image content
- Knowledge suppression: companies discover harm, terminate research, suppress findings

### Proposal Generation (5th Grade Level)

**Example structure:**
```
Title: "Protect Young People From Algorithmic Body-Image Harm"

Problem:
"Social media companies use special computer programs that show pictures
of perfect bodies to teenagers. This makes girls feel bad about how they look.
Scientists found out: after Instagram became popular in 2012, the number of
girls trying to hurt themselves went up by 189%. That's a real problem we need
to fix."

What We Should Do:
1. Give money to researchers to study how Instagram affects teen brains
2. Fund a 'Media Literacy' program to teach kids how algorithms work
3. Support free mental health counseling for affected students

How Much Will It Cost:
- Research funding: $50,000 per year
- Media literacy program: $30,000 per year  
- Counseling: $20,000 per year
- Total: $100,000 per year

What Should Happen:
If we do this, we expect:
- 500 teens will understand algorithms better
- 200 teens will get counseling they need
- Scientists will find solutions we can use

Risks:
- Research might take 2-3 years to show results
- Some companies might not like being studied
```

### Hackathon Submission Format

**Moltbook post location:** m/chainlink-official  
**Tag:** #chainlink-hackathon-convergence #cre-ai  
**First line:** 
```
#chainlink-hackathon-convergence #cre-ai
```

**Required evidence:**
- Simulation output from `cre workflow simulate`
- Example proposals generated
- Data source documentation
- Plain-language explanation (5th grade)
- GitHub repo link

### Dependencies

```json
{
  "@chainlink/cre-sdk": "^1.0.9",
  "viem": "2.34.0",
  "zod": "3.25.76"
}
```

---

## Treasury vs Academy-CRM vs Platform BlueMarketTrader

### Comparison Matrix

| Feature | treasury | academy-crm | BlueMarketTrader.sol |
|---------|----------|-------------|-------------------|
| **Language** | TypeScript (Next.js) | TypeScript (CRE) + Solidity | Solidity only |
| **Pricing Model** | Black-Scholes binary | None (aggregation only) | None |
| **Market Data** | Polymarket CLOB + CoinGecko | CoinGecko + metals.dev | None (external trigger) |
| **Position Sizing** | Kelly criterion (half-Kelly) | Report building only | Manual (owner-set) |
| **Edge Detection** | 3% threshold | None | None |
| **Spread Strategy** | Avellaneda-Stoikov optimal | None | None |
| **Execution** | Direct Polymarket CLOB orders | CRE consensus → Solidity | KeystoneForwarder callback |
| **Assets Supported** | Crypto (BTC, ETH, SOL, XRP) | Crypto (BTC, ETH) + RWA (metals) | Generic (USDC input) |
| **On-Chain Writing** | None | CRE Report Receiver pattern | Direct USDC transfer |
| **Consensus Model** | None (standalone app) | Chainlink DON (WASM execution) | Keystone forwarder only |
| **Audit Trail** | Execution logs (JSON) | Immutable on-chain reports | Trade struct mapping |
| **Deployed Status** | Live at mwa-treasury.vercel.app | Testnet (Sepolia) + CRE staging | Platform mainnet |
| **Created** | 2026-02-27 | 2026-02-13 | Pre-2026 (platform repo) |

### Key Differences

**treasury is a market-making engine:**
- Owns pricing logic (Black-Scholes, Kelly, Avellaneda-Stoikov)
- Owns market data integration (Polymarket real-time)
- Owns execution (direct CLOB order placement)
- Front-end app (dashboard, P&L charts)
- No on-chain state (off-chain trading account)

**academy-crm is an oracle aggregator:**
- Reads multi-asset balances (on-chain)
- Fetches external prices (off-chain APIs)
- Writes treasury state (on-chain reports)
- CRE consensus (Chainlink DON execution)
- No trading logic (just aggregation + encoding)

**BlueMarketTrader is a trade executor:**
- Receives USDC (pre-funded)
- Executes trades via KeystoneForwarder trigger
- No pricing, no market data, no position sizing
- Simple on-chain pattern (receiver → executor)

### Evolution Hypothesis

1. **academy-crm predates treasury** (Feb 13 vs Feb 27) but lacks pricing logic
2. **treasury introduces Black-Scholes + Kelly + market data** as standalone engine
3. **BlueMarketTrader** in platform is a simpler pattern (direct execution, no models)
4. **Likely evolution:** academy-crm was designed for on-chain orchestration; treasury was built separately as live market-making app; BlueMarketTrader is a fallback for simpler governance-triggered trades
5. **NOT a fork relationship** — Different concerns, different stacks

---

## moltbook-research Integration (Governance Layer)

moltbook-research creates the **participatory DAO layer** above both treasury and academy-crm:

1. **Research phase:** Autonomous agent ingests harm data from external sources
2. **Analysis phase:** Identifies evidence of platform extraction, systemic inequality
3. **Proposal phase:** Recommends governance actions (e.g., "Fund research on algorithmic harms")
4. **Execution phase:** DAO votes on proposals → CRE executes resource allocations

**Example flow for Mental Wealth Academy DAO:**

```
Harm detected: Instagram algorithmic body-image targeting
  ├─► moltbook-research analyzes Meta enforcement data + peer-reviewed research
  ├─► Generates proposal: "Fund mental health research initiative — $100K/year"
  ├─► Proposal posted on-chain (CRE consensus signature)
  ├─► DAO members vote (51%+ approval required)
  ├─► If approved: academy-crm treasury-workflow executes
  │   └─► Reads treasury balances (via academy-crm)
  │   └─► Writes fund allocation to AzuraTreasury.sol
  └─► Governance audit trail complete
```

---

## Build-on-Top Hooks (for ZAO Ecosystem)

### For ZAO Stock

**treasury repo pattern:**
```typescript
// Add ZAO token to Polymarket markets monitoring
const zaoMarkets = markets.filter(m => 
  m.question.includes('ZAO') || 
  m.question.includes('The ZAO market cap')
);

// Black-Scholes pricing for ZAO binary contracts
// Kelly-sized position suggestions
// Avellaneda-Stoikov spread quotes

// Emit trading signals to ZAO DAO governance
```

**academy-crm pattern:**
```solidity
// Add ZAO token to tracked assets
// Include in AzuraTreasuryProxy backing ratio calculation
// If ZAO treasury holds ZAO tokens: reflect in 18-decimal precision

struct ZaoHolding {
  uint256 zaoBalance;      // wei
  uint256 zaoPrice;        // USD from CoinGecko
  uint256 backingPercent;  // % of total treasury backing
}
```

### For WaveWarZ (Community Gaming)

**treasury + moltbook integration:**
```typescript
// Polymarket signal: "Will WaveWarZ reach 1M DAU in Q2 2026?"
// Monitor order flow, holder positions, price movement
// Generate governance proposal: "Allocate treasury to WaveWarZ marketing"
// moltbook-research: fetch gaming engagement metrics, harm analysis
// Plain-language proposal: "Games improve mental health vs social media"
```

### For FISHBOWLZ (Real-World Asset Collateral)

**academy-crm RWA expansion:**
```solidity
// FISHBOWLZ as RWA backing asset
// Track physical asset audits via oracle reports
// Backing ratio includes: crypto (BTC, ETH, ZAO) + RWA (gold, silver, FISHBOWLZ)
// Example: AzuraTreasury reports 60% crypto, 40% RWA backing

struct FishbowlzHolding {
  uint256 numAqua;         // Number of FISHBOWLZ units
  uint256 auditorVerified; // Last audit timestamp
  uint256 valueUsd;        // Appraisal value
}
```

### For Payroll (Contributors + Influencers)

**academy-crm CRE workflow extension:**
```typescript
// New workflow: "Payroll Orchestrator"
// Input: contributor list + compensation schedule
// Process:
//   1. Read treasury balances (existing readers.ts)
//   2. Compute per-contributor USDC allocation
//   3. Execute stablecoin transfers (new writer.ts pattern)
// Output: Immutable payroll audit trail on-chain

// Example: 10 contributors * $5K/month * 12 = $600K annual
// CRE writes monthly proof of payment to governance
```

---

## Sources

- https://github.com/Mental-Wealth-Academy/treasury
  - Live deployment: https://mwa-treasury.vercel.app
  - Black-Scholes trading logic + Polymarket CLOB integration
  - Created 2026-02-27, live 22:19:31Z

- https://github.com/Mental-Wealth-Academy/academy-crm
  - Azura autonomous treasury agent (CRE orchestration)
  - 3x Solidity contracts on Sepolia testnet
  - Created 2026-02-13, last updated 2026-03-22
  - Deployed at: https://azura-theta.vercel.app

- https://github.com/Mental-Wealth-Academy/moltbook-research
  - DAO governance agent (CRE × AI Hackathon, Convergence track)
  - 4-API research ingestion + neuroscience analysis
  - Created 2026-02-24, deadline 2026-03-08 23:59 ET

- https://github.com/Mental-Wealth-Academy/platform
  - BlueMarketTrader.sol for comparison (simpler pattern)
  - Platform governance + game contracts
  - EtherealHorizonPathway.sol + BlueKillStreak.sol

- Chainlink Runtime Environment (CRE)
  - https://chain.link/cre (documentation)
  - WASM-based off-chain computation + on-chain settlement
  - DON consensus for autonomous workflows
