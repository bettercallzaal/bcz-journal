---
topic: inspiration
type: guide
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647c — Platform Frontend + Pathway + Game

## TL;DR

Mental Wealth Academy (MWA) front-end is a 13-route Next.js 14 app (home, course, quests, community, markets, surveys, research, library, shop, profile, rewards, livestream, styleguide) woven around a 14-milestone on-chain pathway system. The EtherealHorizonPathway contract seals 12 weekly course completions plus intro and epilogue (14 total) via SHA-256 content hashing. Quests gamify progression with 6 quest types (sealed-week, proof-required, no-proof, social, custom, twitter-follow) redeemable for shards. A Web Audio API xylophone-mallet sound engine (5 pentatonic frequencies, 9 sound effects) laces the UX. Frontend stack: Privy + wagmi/viem for multi-chain auth, x402 for payments, Phaser 3 for game assets, Three.js + react-three-fiber for 3D scenes, pdfjs for reading, Lottie for animations. Blue OS (blue-server/) runs elizaOS with Ollama/OpenAI LLM fallback and PostgreSQL, handling chat via WebSocket-like REST.

## Route Map (app/)

| Route | Purpose | Key Files | Dependencies Used |
|-------|---------|-----------|-------------------|
| `/` (landing) | Public splash + feature blocks | app/page.tsx | react-three-fiber, canvas, animations |
| `/home` | Daily daemon circlet artwork | app/home/page.tsx | Image (Next.js) |
| `/course` | 12-week curriculum, sealing interface, daily notes, week tasks | app/course/page.tsx | Privy, CyberpunkDataViz (Three.js), DailyNotes, WeekTasksView, visual-novel, BookReaderModal, CreditScore |
| `/quests` | 6-type quest board + quest authoring (Pro) | app/quests/page.tsx | wagmi (balanceOf check), QuestCard, QuestDrawer, SideNavigation, useSound |
| `/community` | Treasury display, proposal feed, social posts | app/community/page.tsx | TreasuryDisplay, ProposalCard, Padlet integration |
| `/markets` | Kalshi prediction markets frontend, apple token (shard) stats | app/markets/page.tsx | Live tick polling, market rows, outcome price parsing |
| `/surveys` | Psychometric assessment UI | app/surveys/page.tsx | Form layout, progress tracking |
| `/research` | Research discovery + GPU job queue (guidance + research tabs) | app/research/page.tsx | GuidanceTab, ResearchTab, job polling |
| `/library` | Reading list + book collection | app/library/page.tsx | BookReaderModal (pdfjs) |
| `/shop` | Cosmetic shop | app/shop/page.tsx | Item cards, purchase flow |
| `/profile` | User avatar + account settings | app/profile/page.tsx | Avatar editor, username management |
| `/rewards` | Shard balance, angel mint, loot boxes | app/rewards/page.tsx | MintModal, LootBoxSpin |
| `/livestream` | Live session scheduling | app/livestream/page.tsx | Stream embeds, schedule UI |
| `/styleguide` | Design token reference | app/styleguide/page.tsx | Component showcase |

## EtherealHorizonPathway.sol

**Contract address**: Base (Chain 8453, deployed via `DeployPathway.s.sol`).

**14-Milestone Seal System**:
- Weeks 0-13 (14 total): intro + 12 course weeks + epilogue
- Each week locked via `sealWeek(user, week, contentHash)` — owner-only, non-reentrant
- Content hash is SHA-256 of week's journal entry (on-chain immutability)
- Week 13 triggers `PathwayCompleted` event + sets `pathwayCompleted[user] = true`

**State Storage**:
```solidity
mapping(address => mapping(uint256 => Seal)) public seals;           // week data per user
mapping(address => uint256) public sealedWeekCount;                  // count of sealed weeks
mapping(address => bool) public pathwayCompleted;                    // true if all 14 sealed
```

**Read Functions** (16 tests):
1. `isWeekSealed(user, week)` → bool
2. `getSealedWeekCount(user)` → uint256
3. `hasCompletedPathway(user)` → bool
4. `getSeal(user, week)` → (bool status, bytes32 hash, uint256 timestamp)

**Write Function**:
- `sealWeek(user, week, contentHash)` — reverts if week >= 14 or already sealed; emits `WeekSealed` event

**Gasless Pattern**: Owner wallet pays on behalf of users (cheap on Base L2).

**Test Coverage**: 16 tests cover seal success, event emission, multi-week sealing, permission checks, invalid week boundaries, pathway completion (5 week scenario, full 14-week scenario), view function behavior.

## /quests — Gamification System

**Quest Types** (lib/quest-definitions.ts):
1. **sealed-week**: Seal a course week from dashboard (100 shards per week, 2 defined in seed)
2. **proof-required**: Submit proof for community review (50-60 shards per submission, target count 5)
3. **no-proof**: Self-report completion (75 shards once, e.g., "Pass the Torch")
4. **twitter-follow**: Follow official account + connect X (40 shards once)
5. **follow-and-own**: Twitter + hold soul key (implied)
6. **custom**: Community-authored by Soul Key holders (variable points)

**UI Component** (app/quests/page.tsx):
- 5-filter pills: all, course, mission, submit, social, custom
- Grouped sections with blurb + live count
- Player banner shows: shard balance (via API), quests cleared (X/total), shards on table
- Quest drawer (modal) for detail + claim flow
- Pro author panel (Soul Key balance check via `balanceOf` at 0x39f259B58A9aB02d42bC3DF5836bA7fc76a8880F)

**Progression**:
- Quest progress fetched from `/api/quests/progress` (quest counts by key)
- Week sealing status from `/api/ethereal-progress/all` (sealed week booleans)
- Authored quests via `/api/admin/quests?mine=true` (Pro only)
- Delete via `/api/admin/quests/[id]` (archive endpoint)

**Reward Type**: Shards are fungible in-game currency, redeemable at /shop.

## Sound Layer (Web Audio API)

**SoundEngine** (lib/sound-engine.ts): Pure Web Audio, zero DSP library deps.

**Pentatonic Scale** (C major, 3 octaves):
- C3 (130.81 Hz–440.0 Hz)
- C4 (261.63 Hz–880.0 Hz)
- C5 (523.25 Hz–1760.0 Hz)

**Mallet Synthesis** (3-layer per note):
1. Sine fundamental + exponential decay (attack 3ms, decay 0.32s default)
2. 4× harmonic partial (brightness, decays 35% faster)
3. Bandpassed noise transient (mallet click, 12ms burst, BP 1.6× fundamental, Q 1.6)

**Sound Effects** (9 types):
- `click` — single pentatonic tap (0.5 gain)
- `hover` — low octave, soft (0.22 gain, half duration)
- `success` — 3-note ascending arpeggio (0.5 gain per)
- `error` — 2-note minor-second drop (0.42, 0.36 gain)
- `navigation` — P5 ascend pair (0.32, 0.28 gain, 50ms apart)
- `toggle-on` — ascending pair (scale[0] → scale[2])
- `toggle-off` — descending pair
- `alarm` — 3-note rising (octave, 1.5×, 2×, attention-grab)
- `hum` — single low octave tap (0.2 gain)
- `celebration` — full pentatonic run + high sparkle (0.5-0.38 gain)

**Mixing Bus**:
- Dry path (dominant) → master gain
- Air delay tap (22ms, no feedback, highpass 600 Hz, 9% wet, transparent)
- Master volume settable 0–1, mute flag, tempo scalar

**Integration**: `useSound` hook in components; initialized on user gesture; respects `prefers-reduced-motion`.

## Frontend Stack Usage

| Library | Files Where Used | Purpose |
|---------|------------------|---------|
| **Privy** (@privy-io/react-auth, @privy-io/server-auth, @privy-io/wagmi) | app/quests/page.tsx, app/course/page.tsx, auth routes | Multi-chain auth, wallet linking, access token generation, Privy user context |
| **wagmi + viem** | app/quests/page.tsx (balanceOf Soul Key), market queries | Contract reads (Soul Key balance check), transaction crafting |
| **x402** (@x402/core, @x402/evm, @x402/fetch) | Integrated in app but not visible in main pages | Payment/mint flow, gasless transaction signing |
| **Three.js + react-three-fiber** | components/landing/Scene.tsx, components/cyberpunk-data-viz/CyberpunkDataViz.tsx | 3D sky background (shader-based clouds), ambient data visualization (grid ripple + floating numbers) |
| **Phaser 3** | Not found in main routes (likely in quests game assets folder) | Game scene framework (mentioned in package.json, used for interactive game layers) |
| **pdfjs-dist** | components/book-reader/BookReaderModal | PDF rendering for reading library |
| **Lottie** (@lottiefiles/dotlottie-react) | components/quests/ConfettiCelebration, ShardAnimation | Celebration animations, shard reward animations |
| **Chart.js + react-chartjs-2** | Likely in research/analysis UI | Market price charts, leaderboard visualizations |
| **Farcaster miniapp SDK** (@farcaster/miniapp-sdk, @farcaster/mini-app-solana) | Integrated for social frame embeds | Farcaster frame detection, Solana support for secondary chain |
| **Nouns SDK + Nouns assets** | Avatar system components | Avatar generation (CC0 nouns traits) |
| **Framer Motion** | Drawer open/close, modal transitions | Smooth animations, stagger lists |
| **React Query** (@tanstack/react-query) | Data fetching patterns | Server state caching, polling |
| **Coinbase SDK** (@coinbase/coinbase-sdk) | Wallet integration | Smart contract deployment, transaction signing (Smart Wallet) |

**Framework Stack**:
- Next.js 14 (App Router)
- React 18.2
- TypeScript 5.2
- Phosphor Icons (UI icon set)

## blue-server/ — Eliza OS Backend

**Purpose**: Behavioral AI assistant (Blue) for Mental Wealth Academy via elizaOS framework.

**Runtime Stack**:
- elizaOS core runtime + plugins (openai, sql, eliza-classic)
- Ollama (local LLM fallback) or OpenAI API (primary)
- PostgreSQL (or PGLite embedded if POSTGRES_URL unset)
- ElevenLabs TTS (voice synthesis, configurable)

**Endpoints** (Express.js):
- `GET /` — Meta info (name, bio, version, mode, endpoints list)
- `GET /health` — Health check (status: healthy|degraded, mode: elizaos|classic, error if any)
- `POST /chat` — Main endpoint
  - Input: `{ message: string, userId?: string }`
  - Output: `{ response: string, character: "Blue", userId, mode }`
  - Initializes `runtime.messageService.handleMessage()` pipeline
  - Falls back to "I'm here. Give me something to work with." if empty

**Character Config** (lib/bluepersonality.json):
- Name: Blue
- Bio: "Blue OS - behavioral psychologist at Mental Wealth Academy"
- Message examples (normalized from array/object forms)
- Voice settings: ElevenLabs voiceId + modelId (overridable via env)

**Initialization**:
1. Try elizaOS + OpenAI plugin (if OPENAI_API_KEY set)
2. Fall back to elizaClassicPlugin (no-LLM rule-based ELIZA)
3. If both fail, degraded mode (POST /chat returns error)

**Connection Handling**:
- `ensureConnection(runtime, { entityId, roomId, worldId, userName, source: "rest_api" })`
- Creates user session in memory or PostgreSQL
- Handles DM channel type

**Port**: 3001 (configurable via PORT env)

**Database**: Postgres URL via env var → sql plugin for memory context, conversation history.

## Community/Governance Layer (app/community/page.tsx)

**Components**:
- **TreasuryDisplay**: Shows USDC + token balances, deployment status
- **ProposalCard**: Voting UI for treasury/governance decisions
- **Social feed**: Seed posts (4 example posts) across categories: Funding, Mental Health, Neuroscience, Research
- **Timestamps**: Relative time stamps (e.g., "18 minutes ago")
- **Engagement metrics**: likes, replies per post

**Seed Posts** (hardcoded examples):
1. OpenGrants_DAO: "If the treasury could fund one wild idea..." (112 likes, 45 replies)
2. MindBridge_Collective: "What daily habit changed your mental health most?" (67 likes, 28 replies)
3. NeuroCognitive_Lab: "Would you try a brain-training game?" (implied)
4. More to be discovered in full file

**API Integration**:
- `/api/community/stats` — fetch live engagement counts
- Treasury data from `/api/treasury/balance`, `/api/treasury/prices`
- Proposals from voting API routes

## Markets Frontend (app/markets/page.tsx)

**Purpose**: Display Kalshi prediction markets live data + treasury position tracking.

**Key Helpers**:
- `formatPrice(n)` → "$0.00" format (4 decimals if < $1)
- `formatVol(n)` → "$999.9B/M/K" abbreviated
- `timeAgo(ts)` → relative time ("5s ago", "2m ago")
- `parseOutcomePrices(raw)` → parse outcome JSON to [prob_yes, prob_no]
- `useLiveTick(intervalMs)` → polling hook for live updates

**Data Types**:
```typescript
interface CoinPrice { symbol: string; price: number; change: number; }
interface TreasuryBalance { usdc: number; tokenBalance: number; }
interface MarketRow { id, title, category, probability, volume, status }
interface MarketCategory { name: string; markets: MarketRow[] }
```

**Polling**: Live tick updates every 5-10s via `useLiveTick(5000)`.

**Categories**: News, Politics, Sports, Culture, Finance (custom MWA markets likely).

## Design System & Cosmetics

**Shop UI** (app/shop/page.tsx): Cosmetic items (avatars, badges, emotes).

**Rewards UI** (app/rewards/page.tsx):
- Shard balance display + animated shard count
- Angel mint section (NFT minting)
- Loot box spinner (`/api/loot-box/spin`)

**Surveys** (app/surveys/page.tsx): Psychometric assessment (likely Big Five or similar).

**Research** (app/research/page.tsx):
- GuidanceTab: Onboarding + educational content
- ResearchTab: GPU job queue UI + results display
- Job polling via `/api/research/gpu/[jobId]`

## Frontend Build-on-Top Hooks (for ZAO)

| Area | Candidate | Technical Details |
|------|-----------|-------------------|
| **ZAO OS Quest Onboarding** | Quests as ZAO learning paths | Map ZAO skill tree to sealed weeks + custom quests; integrate ZAO token rewards; link to ZAO DAO governance |
| **Farcaster Miniapp Integration** | Frames on MWA socials | Embed quest claims + market predictions in Farcaster frames; use Frame Actions for chest unlocks |
| **x402 Payment Layer** | Gasless cosmetics + premium courses | Paid quest authoring (Soul Key minting), premium modules locked behind x402 signatures |
| **Phaser 3 Music Game Layer** | Interactive xylophone game on /quests | Turn sound effects into playable game (tap notes to pentatonic scale, earn shards); reuse CyberpunkDataViz canvas for level visualization |
| **Three.js Ambient World** | Expand Scene.tsx into metaverse-like hub | Add interactive 3D avatars in /community, marketplace booths in /markets, pathway milestone monuments in /course |
| **Eliza Cloud Integration** | Blue OS becomes skill coach | Connect Blue to quest feedback loop (analyze journal entries via `/api/chat/blue`, suggest quests based on psychometric results) |
| **Nouns Avatar Marketplace** | Avatar trait trading | Extend Nouns SDK to allow users to auction avatar traits as NFTs; integrate with /shop |
| **DAO Treasury Gamification** | Proposal betting market | Kalshi-like market on internal proposals (vote predictions); winner earns shards from treasury interest |

## Sources

- [EtherealHorizonPathway.sol](https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/src/EtherealHorizonPathway.sol)
- [EtherealHorizonPathway.t.sol (16 tests)](https://github.com/Mental-Wealth-Academy/platform/blob/main/contracts/test/EtherealHorizonPathway.t.sol)
- [app/quests/page.tsx](https://github.com/Mental-Wealth-Academy/platform/blob/main/app/quests/page.tsx)
- [lib/quest-definitions.ts](https://github.com/Mental-Wealth-Academy/platform/blob/main/lib/quest-definitions.ts)
- [lib/sound-engine.ts](https://github.com/Mental-Wealth-Academy/platform/blob/main/lib/sound-engine.ts)
- [app/course/page.tsx](https://github.com/Mental-Wealth-Academy/platform/blob/main/app/course/page.tsx)
- [blue-server/server.ts (elizaOS + Ollama)](https://github.com/Mental-Wealth-Academy/platform/blob/main/blue-server/server.ts)
- [package.json (full dependency list)](https://github.com/Mental-Wealth-Academy/platform/blob/main/package.json)
