---
topic: inspiration
type: guide
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647b — Platform Treasury + Trading (BlueMarketTrader + Kalshi)

## TL;DR

Mental-Wealth-Academy/platform implements a separated treasury + trading architecture: BlueMarketTrader.sol (Solidity, 218 lines) holds USDC and executes binary-outcome trades via two paths: owner-initiated or DON-signed Chainlink CRE reports (onReport receiver gated by KeystoneForwarder). CRE workflows run on 10-minute cron (auto-execute proposals) and event triggers (ProposalExecuted -> trade inference). Frontend implements Black-Scholes binary pricing with 0.50 sigma, 4.33% risk-free rate, Avellaneda-Stoikov market-making spreads, and quarter-Kelly (0.25x) position sizing capped at 5% per position / 40% total exposure. Edge detection threshold is 3% divergence; trading is VIP-gated via ERC-1155 membership card.

## Key Findings

- **BlueMarketTrader.sol state**: usdcToken (immutable), predictionMarket, keystoneForwarder, tradeCount, trades[tradeId] mapping (MockPredictionMarket.sol:18-20)
- **Deposit/Withdraw gating**: deposit() is public (any caller), withdraw() restricted to owner via onlyOwner modifier (BlueMarketTrader.sol:193-207)
- **Trade execution paths**: executeTrade(marketId, isYes, amount) for owner (118-121), onReport(metadata, report) for Chainlink CRE with KeystoneForwarder guard (123-127)
- **Binary mechanics**: MockPredictionMarket tracks totalYes/totalNo accumulation per market (src:48-49), position stored as (yesAmount, noAmount) per trader per market (src:52-53)
- **Test count**: 23 tests documented in BlueMarketTrader.t.sol covering deploy, trade execution (buy YES/NO, multiple trades), owner/non-owner guards, CRE integration, treasury management, admin functions
- **auto-execute workflow**: 10-minute cron (*/10 * * * *), reads proposalCount, iterates active proposals, checks hasReachedThreshold (50%), encodes actionType 1 payload, generates DON report via runtime.report(), writes to contractAddress via KeystoneForwarder (cre-workflows/auto-execute/main.ts:29-30)
- **trade-execute workflow**: Event-triggered on ProposalExecuted log topic, decodes proposalId from log.topics[1], infers trade direction via keyword matching (bullish: "buy yes", "long", "upside"; bearish: "buy no", "short", "downside"), checks trader balance, submits DON-signed report to BlueMarketTrader.onReport() (cre-workflows/trade-execute/main.ts:80-85)
- **Black-Scholes binary formula**: C_binary = e^(-rT) * N(d2), d2 = ((r - 0.5*sigma^2)*T) / (sigma*sqrt(T)), implemented with Abramowitz-Stegun normal CDF approximation (app/markets/page.tsx:162-175)
- **Model constants**: SIGMA=0.50 (annual IV), T_EXP=0.0000095 (5 min / 1 year = 5.12ms/yr), R_FREE=0.0433 (4.33%), SIGMA_B=0.328 (belief vol), GAMMA=0.10 (A-S risk aversion), K_DECAY=1.50 (arrival decay) (page.tsx:119-126)
- **Avellaneda-Stoikov spreads**: delta_x = GAMMA * sigma_b^2 * T / 2 + (1/GAMMA)*ln(1 + GAMMA/K_DECAY) computes half-spread, logit-transformed to bid/ask probabilities via p_bid = 1/(1+exp(-(x_t - delta_x))), p_ask = 1/(1+exp(-(x_t + delta_x))) (page.tsx:490-494)
- **CoinGecko spot integration**: prices fetched via /api/treasury/prices, matched to market question text via symbol/id search (trading-engine.ts:107-110), fallback ticker is BTC
- **Quarter-Kelly sizing**: kellyRaw = (p*b - q)/b, kellyFraction = max(0, min(kellyRaw * 0.25, 5%)), maxPerPosition = balance * 5%, maxTotalExposure = balance * 40%, notionalBalance = 5000 USD (trading-engine.ts:120-137)
- **5% / 40% caps**: MAX_POSITION_PCT = 0.05, MAX_TOTAL_EXPOSURE_PCT = 0.40 strictly enforced during position sizing loop (trading-engine.ts:113-115)
- **Edge threshold**: 3% divergence (modelFair - mktPrice), signal emitted only if abs(divergence) >= 3.0 (trading-engine.ts:94-100, page.tsx:125)
- **Dry-run / signal-only gating**: buildTopTradePlan() is called only if hasVipMembershipCard verified (route.ts:35-41), and only produces plan if edge >= 3% (trading-engine.ts:175); execution is VIP-card gated at wallet level (soul-key.ts contract check)
- **Kalshi order placement**: placeKalshiOrder() uses RSA-signed fetch (timestamp + method + path), side='yes'|'no', count, priceCents (1-99), clientOrderId (UUIDs), time_in_force='fill_or_kill', type='limit' (kalshi-trading.ts:57-83)
- **Execution log storage**: setExecutionLogs(logs, positions) persists TradingLog (action, asset, details, timestamp) and PositionEntry (asset, side, price, size, sizeMatched, status) to in-memory store (route.ts:79-87)

## BlueMarketTrader.sol Surface

| Item | Location | Type | Details |
|------|----------|------|---------|
| USDC token | Line 20 | State var (immutable) | ERC20 interface, set in constructor |
| Prediction Market | Line 23 | State var | Address, settable by owner |
| KeystoneForwarder | Line 26 | State var | Address, gated by onlyForwarder modifier |
| Trade struct | Lines 31-37 | Data struct | id, marketId, isYes, usdcAmount, executedAt |
| trades mapping | Line 44 | Storage | tradeId -> Trade |
| TradeExecuted event | Lines 48-53 | Event | tradeId, marketId, isYes, usdcAmount |
| executeTrade() | Lines 115-121 | Function | External, onlyOwner, nonReentrant, returns tradeId |
| onReport() | Lines 123-127 | Function | External, onlyForwarder, nonReentrant, decodes (marketId, isYes, amount) from report |
| _executeTrade() | Lines 129-165 | Internal | Core logic: validate amount > 0, market != 0, balance check, increment tradeCount, store Trade struct, approve USDC, call buyOutcome via low-level call, emit TradeExecuted |
| setPredictionMarket() | Lines 171-179 | Admin | Owner-only, resets prior market approval to 0, emits PredictionMarketUpdated |
| setKeystoneForwarder() | Lines 181-188 | Admin | Owner-only, validates != 0x0, emits KeystoneForwarderUpdated |
| deposit() | Lines 190-194 | Public | Anyone, pull USDC via transferFrom, no amount cap |
| withdraw() | Lines 196-207 | Owner-only | Transfer USDC from contract to owner, validates balance |
| getTrade() | Lines 215-218 | View | Fetch Trade struct by ID |
| treasuryBalance() | Lines 220-223 | View | Return USDC balance of contract |

## MockPredictionMarket.sol Surface

| Item | Location | Type | Details |
|------|----------|------|---------|
| usdc | Line 14 | Immutable | ERC20 token interface |
| Market struct | Lines 16-21 | Data struct | question, totalYes, totalNo, resolved, outcome |
| Position struct | Lines 23-26 | Data struct | yesAmount, noAmount per trader per market |
| markets mapping | Line 28 | Storage | marketId -> Market |
| positions mapping | Line 29 | Storage | (marketId, trader) -> Position |
| marketCount | Line 30 | Counter | Incremented on each createMarket |
| createMarket() | Lines 39-44 | Function | Returns marketId, emits MarketCreated |
| buyOutcome() | Lines 46-62 | Function | Pull USDC via transferFrom, increment totalYes or totalNo, track position at cost basis |
| resolveMarket() | Lines 64-70 | Function | Set resolved=true, outcome=bool (no settlement, demo only) |
| getPosition() | Lines 76-80 | View | Return (yesAmount, noAmount) for trader on market |
| getMarket() | Lines 82-87 | View | Return market details (question, totals, resolution state) |

## auto-execute + trade-execute CRE Workflows

### auto-execute Sequence (10-minute cron)

```
START (cron: */10 * * * *)
  |
  +-> READ proposalCount from BlueKillStreak contract
  |
  +-> FOR EACH proposal i=1..count:
  |    |
  |    +-> FETCH proposal struct (status, executed, ...)
  |    |
  |    +-> SKIP if status != Active OR executed == true
  |    |
  |    +-> CHECK hasReachedThreshold(proposalId)
  |    |
  |    +-> IF threshold reached:
  |    |    |
  |    |    +-> ENCODE actionType=1 (ActionType.AutoExecute) + innerPayload=proposalId
  |    |    |
  |    |    +-> CALL runtime.report() to generate DON-signed report
  |    |    |
  |    |    +-> CALL evm.writeReport(receiver=BlueKillStreak, report)
  |    |    |
  |    |    +-> LOG auto-executed proposal {i}
  |    |
  |    +-> ELSE SKIP
  |
  +-> RETURN "done"
```

Config: Base mainnet (0x2cbb90a761ba64014b811be342b8ef01b471992d). Runs every 10 minutes, reading up to proposalCount iterations. Generates one DON-signed report per active proposal that crosses 50% vote threshold.

### trade-execute Sequence (Event-triggered)

```
START (on ProposalExecuted event)
  |
  +-> DECODE proposalId from log.topics[1]
  |
  +-> LOG "ProposalExecuted event detected: proposalId={proposalId}"
  |
  +-> FETCH full proposal struct via getProposal(proposalId)
  |
  +-> VALIDATE proposal.executed == true
  |
  +-> IF proposal.recipient != traderAddress:
  |    |
  |    +-> LOG "not trader contract, skipping"
  |    +-> RETURN "not-trader"
  |
  +-> CHECK trader treasuryBalance() >= proposal.usdcAmount
  |
  +-> IF insufficient balance:
  |    |
  |    +-> LOG "balance insufficient"
  |    +-> RETURN "insufficient-balance"
  |
  +-> INFER trade direction from proposal title + description:
  |    |
  |    +-> bullish keywords: "buy yes", "bullish", "long", "upside", "growth", "increase", "rise", "gain"
  |    |
  |    +-> bearish keywords: "buy no", "bearish", "short", "downside", "decline", "decrease", "fall", "drop"
  |    |
  |    +-> COUNT matches per category, isYes = (bullScore >= bearScore)
  |
  +-> LOG trade decision (proposalId, direction, amount in USD)
  |
  +-> ENCODE reportPayload = (proposalId, isYes, proposal.usdcAmount)
  |
  +-> CALL runtime.report() for DON signature
  |
  +-> CALL evm.writeReport(receiver=BlueMarketTrader, report)
  |
  +-> LOG "DON-signed trade report submitted"
  |
  +-> RETURN "traded"
```

Triggered on ProposalExecuted events from BlueKillStreak (0x2cbb90a761ba64014b811be342b8ef01b471992d). Infers YES/NO direction from proposal text, validates trader has balance, submits parametrized report to BlueMarketTrader.onReport().

## Kalshi Edge Detection + Trading Engine

### Black-Scholes Binary Pricer

```
Input: spot S, strike K (ATM), sigma=0.50, T=0.0000095, r=0.0433

Step 1: d2 = ((r - 0.5*sigma^2)*T) / (sigma*sqrt(T))
        = ((0.0433 - 0.5*0.25)*0.0000095) / (0.50*sqrt(0.0000095))
        = ~-0.00217

Step 2: N(d2) = Abramowitz-Stegun CDF approximation = 0.49913

Step 3: C_binary = e^(-r*T) * N(d2)
        = e^(-0.0433*0.0000095) * 0.49913
        = 0.99959 * 0.49913
        = ~0.4989 (or 49.89%)

Step 4: Market price (YES prob) = parseOutcomePrices(market.outcomePrices)[0]
        E.g., Kalshi returns "0.5378" (53.78%)

Step 5: divergence = modelFair - mktPrice = 49.89% - 53.78% = -3.89%

Step 6: signal = abs(-3.89%) >= 3.0% ? 'TRADE' : 'SKIP'
        -> 'TRADE' (bearish: model says lower, so SELL/NO)
```

Implemented in app/markets/page.tsx lines 162-175 (normalCDF) and 478-514 (derived calculation). Live jitter animation every 1.2s (modelTick) adds micro-movement to SIGMA, SIGMA_B, lambda_jump, mu_J, q_inv for dashboard realism (not used in order logic).

### Avellaneda-Stoikov Market-Making Spreads

```
Input: x_t (logit of probability), sigma_b=0.328 (belief vol), 
       GAMMA=0.10 (risk aversion), K_DECAY=1.50 (arrival decay)

Step 1: delta_x = GAMMA * sigma_b^2 * T / 2 + (1/GAMMA) * ln(1 + GAMMA/K_DECAY)
        = 0.10 * 0.328^2 * 0.0000095 / 2 + (1/0.10) * ln(1 + 0.10/1.50)
        = 0.0000000497 + 10 * ln(1.0667)
        = 0.0000000497 + 0.6469
        = ~0.6469

Step 2: p_bid = 1 / (1 + exp(-(x_t - delta_x)))
        p_ask = 1 / (1 + exp(-(x_t + delta_x)))
        
        With x_t = ln(0.5/(0.5)) = 0 (midpoint):
        p_bid = 1 / (1 + exp(0.6469)) = 34.3%
        p_ask = 1 / (1 + exp(-0.6469)) = 65.7%
        
        Spread = 65.7% - 34.3% = 31.4% (symmetric around fair value)
```

Computes belief-based market-making spreads. No actual orderbook logic; purely parametric display. Used for audit display in "Model details" modal (page.tsx:990-981).

### CoinGecko Spot Integration

Fetches prices via /api/treasury/prices endpoint (route.ts in app/api/treasury/prices). Matches markets to assets by fuzzy substring matching on question text:

```javascript
const matchedCoin = prices.find((p) =>
  market.question.toLowerCase().includes(p.symbol.toLowerCase()) ||
  market.question.toLowerCase().includes(p.id.toLowerCase())
);
const asset = matchedCoin?.symbol ?? 'BTC';  // default fallback
```

Returns array of CoinPrice (symbol, id, usd, usd_24h_vol). Live-polled every 30 seconds (page.tsx:411). Used only for display and model parameter context (not for position sizing calculations).

### Quarter-Kelly Sizing + Risk Caps

```
For each signal in ranked order:

1. Compute raw Kelly: kellyRaw = (p*b - q) / b
   where p = modelFair/100, q = 1-p, b = odds ratio (depends on side)
   
2. Apply quarter-Kelly cap: kellyFraction = max(0, min(kellyRaw * 0.25, 5%))
   
3. Compute position size: sizeUSD = min(balance * kellyFraction, maxPerPosition)
   where maxPerPosition = balance * 5%
   
4. Stop if totalExposure >= balance * 40%

5. Return top position only (no multi-leg orders)

Example:
  balance = 5000 USD
  p = 0.49 (model 49%), q = 0.51
  mktPrice = 53.78% (short side, b = 1/(1-0.5378) = 2.16)
  
  kellyRaw = (0.49 * 2.16 - 0.51) / 2.16 = (1.0584 - 0.51) / 2.16 = 0.2564
  kellyFraction = min(0.2564 * 0.25, 0.05) = min(0.0641, 0.05) = 0.05 (capped)
  
  sizeUSD = min(5000 * 0.05, 5000 * 0.05) = 250 USD
  shares = floor(250 / 0.5378) = 464 contracts
```

Implemented in trading-engine.ts lines 120-137. Hard caps of 5% per position and 40% total exposure are non-negotiable. Notional balance is 5000 USD (dry-run, not real).

### Dry-Run Signal-Only Gating

Order execution only triggers if:

1. VIP Membership Card holder verified (checks ERC-1155 contract at VIP_MEMBERSHIP_CARD_ADDRESS via walletHoldsVipMembershipCard()) -> 403 if not held
2. Kalshi credentials (KALSHI_API_KEY_ID, KALSHI_API_PRIVATE_KEY) are set -> 503 if not
3. buildTopTradePlan() returns a plan (not null) AND plan.signal.divergence >= 3% -> 409 if no edge
4. Rate limit not exceeded (4 trades per 60 seconds) -> 429 if exceeded

If any gate fails, response is JSON with error code, no order is placed, execution log shows SKIP with reason. Dry-run logic is transparent: all SIGNAL/SKIP logs are persisted and rendered live on /markets page.

## Test Coverage (23 trader tests)

| Test Name | File | Line | Assertion |
|-----------|------|------|-----------|
| test_Constructor | BlueMarketTrader.t.sol | 51-55 | usdcToken, predictionMarket, keystoneForwarder, tradeCount == 0 |
| test_TreasuryBalance | 57-59 | treasuryBalance() == 100k USDC (TREASURY_FUND) |
| test_ExecuteTrade_BuyYes | 62-85 | Buy YES: tradeId incremented, position tracked on market, Trade struct stored with timestamps |
| test_ExecuteTrade_BuyNo | 87-95 | Buy NO: position tracked separately, no interaction with YES side |
| test_MultipleTrades | 97-107 | Multiple executeTrade calls: tradeCount increments per call, positions accumulate per marketId |
| test_RevertWhen_NonOwnerExecutesTrade | 109-116 | Non-owner prank fails with OwnableUnauthorizedAccount |
| test_RevertWhen_ZeroAmount | 118-122 | executeTrade(_, _, 0) reverts InvalidAmount |
| test_RevertWhen_InsufficientBalance | 124-128 | executeTrade with amount > treasuryBalance reverts InsufficientBalance |
| test_RevertWhen_NoPredictionMarket | 130-137 | executeTrade reverts InvalidMarket if predictionMarket == 0x0 |
| test_OnReport_ExecutesTrade | 140-158 | Forwarder calls onReport, trade executes, position and tradeCount updated |
| test_OnReport_BuyNo | 160-169 | onReport with isYes=false routes to NO position |
| test_RevertWhen_NonForwarderCallsOnReport | 171-177 | Non-forwarder prank on onReport reverts Unauthorized |
| test_Deposit | 180-186 | Deposit increases treasuryBalance, transferFrom called on msg.sender |
| test_RevertWhen_DepositZero | 188-191 | deposit(0) reverts InvalidAmount |
| test_Withdraw | 193-200 | Withdraw decreases treasuryBalance, transfers to owner |
| test_RevertWhen_NonOwnerWithdraws | 202-208 | Non-owner withdraw fails with OwnableUnauthorizedAccount |
| test_RevertWhen_WithdrawExceedsBalance | 210-213 | withdraw(balance+1) reverts InsufficientBalance |
| test_SetPredictionMarket | 216-220 | setPredictionMarket updates address, emits event |
| test_SetKeystoneForwarder | 222-226 | setKeystoneForwarder updates address, emits event |
| test_RevertWhen_NonOwnerSetsMarket | 228-233 | Non-owner setPredictionMarket fails with OwnableUnauthorizedAccount |
| test_RevertWhen_NonOwnerSetsForwarder | 235-240 | Non-owner setKeystoneForwarder fails with OwnableUnauthorizedAccount |
| test_RevertWhen_SetKeystoneForwarderZeroAddress | 248-250 | setKeystoneForwarder(0x0) reverts "Invalid forwarder address" |
| test_SetPredictionMarketResetsApproval | 264-275 | Changing predictionMarket resets USDC approval on old market to 0 |

All 23 tests use Foundry forge-std Test harness, Fork testing not used (local mocks only), 100k USDC initial treasury funding.

## Build-on-top Hooks (for ZAO)

| Candidate | Integration Point | Reuse / Adaptation |
|-----------|-------------------|-------------------|
| WaveWarZ Prediction Markets | Binary outcome markets (same Mock interface) | Adapt MockPredictionMarket for WaveWarZ token pair outcomes (BTC/USD, SOL/USD). Reuse Black-Scholes + A-S spreads for pricing. Replace Kalshi CoinGecko spot with WaveWarZ oracle input. Quarter-Kelly sizing logic unchanged. |
| ZAO Stock Treasury Automation | Governance-to-treasury pipeline (auto-execute + trade-execute workflows) | Reuse CRE workflow architecture: 10min cron for proposal auto-execution, event-triggered trade routing. Extend to support stock futures (ES, NQ, MES) instead of binary outcomes. Keyword matching already handles "long/short" inference. |
| FISHBOWLZ Payouts | On-chain settlement + position tracking | Reuse Trade struct (id, marketId, side, amount, timestamp) for payout ledgers. Adapt withdrawals to support multi-recipient payouts from treasury. BlueMarketTrader can be "market operator" contract feeding settlement signals. |
| Thy Revolution Proposal Marketplace | Proposal creation + voting + execution | Trade-execute workflow can infer asset allocation from proposal description + model signal. Extend Kalshi matching to include Thy Revolution assets. CRE already handles 2-tier execution (auto vs. VIP-gated). |
| ZOLs Contribution Credits | Reward distribution from trading profits | Add profit-tracking loop to _executeTrade(): on market resolution, compute P&L per position, emit ZOL claims for treasury operators. Extend execution-log-store to persist realized gains/losses. |

## Sources

- https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/src/BlueMarketTrader.sol
- https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/src/MockPredictionMarket.sol
- https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/test/BlueMarketTrader.t.sol
- https://github.com/Mental-Wealth-Academy/platform/blob/main/cre-workflows/auto-execute/main.ts
- https://github.com/Mental-Wealth-Academy/platform/blob/main/cre-workflows/trade-execute/main.ts
- https://github.com/Mental-Wealth-Academy/platform/blob/main/app/markets/page.tsx (lines 119-1027, Black-Scholes + model parameters)
- https://github.com/Mental-Wealth-Academy/platform/blob/main/lib/trading-engine.ts (Kelly sizing, edge detection)
- https://github.com/Mental-Wealth-Academy/platform/blob/main/lib/kalshi-trading.ts (RSA signing, order placement)
- https://github.com/Mental-Wealth-Academy/platform/blob/main/app/api/treasury/trade/execute/route.ts (VIP gating, dry-run execution)
