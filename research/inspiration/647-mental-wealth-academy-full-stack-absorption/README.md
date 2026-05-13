---
topic: inspiration
type: market-research
status: research-complete
last-validated: 2026-05-13
related-docs: 56, 149, 234, 245, 421
tier: DISPATCH
staging-for: bettercallzaal/zaoos
---

# 647 — Mental Wealth Academy Full-Stack Absorption

> **Goal:** Absorb every public repo, contract, workflow, and surface across the Mental-Wealth-Academy GitHub org. Map overlap with The ZAO. Identify concrete "build-on-top" deliverables Zaal can ship and show James (sole MWA dev) to open a collaboration door.

## TL;DR

Mental Wealth Academy is a solo-built (James Marsh / @jqmwa) micro-university DAO running on Base + Optimism. Thesis: **psychology-first decentralized education** with an AI agent ("Blue") that scores funding proposals on a tamper-proof Chainlink DON before community vote. 15 public repos, 70 Solidity tests, 3 CRE workflows live, contract `0x2cbb90...992d` deployed Base mainnet. Stack is overlap-heavy with The ZAO: Privy multi-chain auth, Farcaster miniapp SDK, x402 payments, Three.js, Phaser 3, Eliza agent framework. The biggest reusable primitives are (a) the **Blue 6-dim scoring rubric + level→weight veto model**, (b) the **Black-Scholes binary pricer + Quarter-Kelly sizing pipeline** for Kalshi/Polymarket, and (c) the **EtherealHorizonPathway 14-milestone SHA-256 seal system** for gamified curricula. The ZAO can ship a music-and-artist-centered fork of two or three of these surfaces in 1-3 weeks.

## Org Profile

| Field | Value |
|---|---|
| Org | [Mental-Wealth-Academy](https://github.com/Mental-Wealth-Academy) |
| Created | 2024-01-10 (GitHub org), platform repo 2025-12-05 |
| Lead | James Marsh / "Jhinn Bay" — [@jqmwa](https://github.com/jqmwa), Farcaster @jinn (1928 followers), psychology grad + UX/product designer |
| Funding | Solo-funded since inception. Optimism RetroPGF applicant Jan 2026 (501-recipient cohort context). |
| Public repos | 15 |
| Tagline | "Investing in the capital of the human mind, with the heart of tomorrow." |
| Domain | mentalwealthacademy.world |
| X | @MentalWealthDAO |
| License | Apache-2.0 across all platform code |
| Governance contract | `0x2cbb90a761ba64014b811be342b8ef01b471992d` (Base) |
| NFT contract | `0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f` (Base, "Academic Angels", minted by survey-iq) |

## Architecture in One Diagram

```
                                 +-------------------------+
                                 |   Member (Privy auth,   |
                                 |   Coinbase SDK, Farc.)  |
                                 +------------+------------+
                                              |
        +-----------------+-----------------+ | +-----------------+----------------+
        | /surveys (IQ)   | /quests (Phaser)| | | /community (DAO)| /markets       |
        | mints Angels    | 14 seals SHA-256| | | Blue scoring    | Kalshi+BS+Kelly|
        +--------+--------+--------+--------+ | +-----+------+----+-------+--------+
                 |                 |          |       |      |            |
                 v                 v          v       v      v            v
        +--------+-----------------+----------+-------+------+------------+-------+
        |              On-chain (Base mainnet)                                    |
        |  BlueKillStreak  •  BlueMarketTrader  •  EtherealHorizonPathway        |
        |  AcademicAngels (ERC-1155)            •  MockPredictionMarket          |
        +--------+-------------+----------------------+--------------------+-----+
                 ^             ^                      ^                    ^
                 |             |                      |                    |
        DON-signed reports via KeystoneForwarder (Chainlink CRE)           |
                 |             |                      |                    |
        +--------+-----+-------+--------+ +-----------+-------+   +--------+-----+
        | blue-review  | auto-execute   | | trade-execute     |   | Vercel cron  |
        | (event)      | (cron 10min)   | | (event)           |   | Kalshi scan  |
        +------+-------+----------------+ +---------+---------+   +--------------+
               |                                    |
               v                                    v
        +------+--------+                  +--------+--------+
        | Eliza Cloud   |                  | BlueMarketTrader|
        | (6-dim score) |                  | onReport()      |
        +---------------+                  +-----------------+

        Side surfaces (separate repos):
        - genetics (browser-native SNP, 110k+ variants, WASM SQLite)
        - research (Lanczos+Lentz exact p-values, Chart.js)
        - Pocket-World / Azure World (Flask+Vue, Zep GraphRAG, OASIS multi-agent sim)
        - treasury (Polymarket CLOB, Black-Scholes mature fork)
        - academy-crm (Azura oracle agent, 6 assets, Sepolia testnet)
        - moltbook-research (Chainlink CRE x AI hackathon)
        - knowledge / wellness-directory / brand-kit / Halftone / mwa-swift / milady (upstream sync)
```

## Repo Inventory (15)

| # | Repo | Lang | Purpose | Live | Risk |
|---|------|------|---------|------|------|
| 1 | [platform](https://github.com/Mental-Wealth-Academy/platform) | TS+Sol | Main academy: LMS + governance + markets | mentalwealthacademy.world | Active, prod |
| 2 | [Pocket-World](https://github.com/Mental-Wealth-Academy/Pocket-World) | Python | Multi-agent sim, doc→futures | azure-world.vercel.app | Active |
| 3 | [treasury](https://github.com/Mental-Wealth-Academy/treasury) | TS | Standalone treasury w/ Polymarket | mwa-treasury.vercel.app | Active |
| 4 | [genetics](https://github.com/Mental-Wealth-Academy/genetics) | TS | Browser-native SNP lab | mwa-genetics.vercel.app | Active |
| 5 | [research](https://github.com/Mental-Wealth-Academy/research) | TS | Stats workbench | mwa-research.vercel.app | Active |
| 6 | [academy-crm](https://github.com/Mental-Wealth-Academy/academy-crm) | Sol | Azura oracle, CRE for payroll/RWA | testnet | Sepolia only |
| 7 | [moltbook-research](https://github.com/Mental-Wealth-Academy/moltbook-research) | TS | Hackathon DAO governance agent | n/a | Hackathon |
| 8 | [knowledge](https://github.com/Mental-Wealth-Academy/knowledge) | md | LLM aggregation hub | n/a | Docs |
| 9 | [survey-iq-initiation](https://github.com/Mental-Wealth-Academy/survey-iq-initiation) | TS | Psychometric onboarding + NFT mint | live | Active |
| 10 | [wellness-directory](https://github.com/Mental-Wealth-Academy/wellness-directory) | TS | Provider search | academyos.vercel.app | Active |
| 11 | [brand-kit](https://github.com/Mental-Wealth-Academy/brand-kit) | md | Brand assets | n/a | Static |
| 12 | [Halftone](https://github.com/Mental-Wealth-Academy/Halftone) | TS | Halftone image processor (internal content gen) | n/a | Tool |
| 13 | [mwa-swift](https://github.com/Mental-Wealth-Academy/mwa-swift) | Swift | iOS native (3 tabs) | not on App Store | Early |
| 14 | [milady](https://github.com/Mental-Wealth-Academy/milady) | TS | **Upstream sync of milady-ai/milady**, not original | n/a | Fork |
| 15 | [.github](https://github.com/Mental-Wealth-Academy/.github) | md | Org profile | n/a | Docs |

## Cross-Cutting Themes

### 1. AI as a voting member, not a tool
Blue is not "an LLM that suggests" — Blue is a **token-weighted governance participant** whose voting weight (10/20/30/40%) is set by a DON-signed AI score from Eliza Cloud, and whose level 0 is an outright veto. The novelty isn't AI scoring; it's making the score tamper-proof via Chainlink's Decentralized Oracle Network signature aggregation, so no single server (not even James's) can fake the score. This collapses the "AI alignment in DAOs" problem from a social one to a cryptographic one.

### 2. Chainlink CRE as governance backbone
3 production workflows: `blue-review` (event-triggered AI score), `auto-execute` (10-min cron, 50% threshold sweep), `trade-execute` (event-triggered, infers YES/NO from proposal text using 8 bullish + 8 bearish keyword sets). CRE is **only used where DON signatures gate on-chain state changes** — the market scanner is a Vercel cron because it's signal-only.

### 3. Privacy-first compute by default
genetics: 110k+ SNPs processed in-browser via WebAssembly SQLite + Comlink Web Workers; DNA never uploaded. research: exact p-values via Lanczos gamma + Lentz continued fraction (3e-14 tolerance, 200-iter cap) entirely client-side. survey-iq: psychometric scoring done client-side before mint. **Pattern: heavy compute moves to the browser, on-chain layer only stores attestations.**

### 4. Gamified curriculum with on-chain proof
EtherealHorizonPathway = 14-milestone SHA-256 sealed pathway (intro + 12 weeks + epilogue). 6 quest types: sealed-week, proof-required, no-proof, social, custom, twitter-follow. Custom Web Audio synthesizer (3-layer per-note: sine fundamental + 4× harmonic + bandpassed noise click) replaces typical reverb-laden game UX with xylophone mallet tones — a deliberate aesthetic choice signaling "calm focus, not gambling dopamine."

### 5. Pluripotent agent stack
ElizaOS (Blue, Azura), Anthropic Claude (chat + research narratives), OpenAI-compatible LLMs (Pocket-World), milady (privacy-first agent framework, synced from upstream). James is **vendor-promiscuous on inference, vendor-locked on infrastructure** (Chainlink, Base, Optimism, Vercel).

### 6. Solo-dev velocity
70 Foundry tests, 13 frontend routes, 3 CRE workflows, 5 live Vercel deployments, 6 contracts, 15 repos — all from one author with ~5 months on platform. May 13 commit burst (~15 commits in one day, all UX polish on /quests) shows the project is in "make it feel good" phase, not "ship more features" phase. This is the moment to approach.

## The Critical Lens

**Strengths:**
- Genuinely novel DON-gated AI governance (not just "Snapshot + ChatGPT review")
- Strong tech taste: Foundry not Hardhat, viem not ethers, Privy not RainbowKit
- Privacy primitives are real, not theater
- Apache-2.0 across the board — fork-friendly

**Weaknesses / risks:**
- Single point of failure: one dev, one wallet, one Eliza Cloud API key
- 1 follower on org account, 1928 on lead's personal Farcaster — distribution is the unsolved problem
- academy-crm still Sepolia (not mainnet), moltbook is hackathon code, milady is an upstream sync — surface is broader than depth in places
- Black-Scholes for short-dated political/economic binaries is theoretically wrong (binary outcomes are not log-normal); the model is a heuristic, not pricing-correct. Quarter-Kelly + 3% edge threshold mitigates, but a sophisticated counterparty would clean this out.
- Pocket-World rebranded from MiroFish (Shanda Group origin) — provenance worth flagging if integrating

**Counterpoint for ZAO:** The "AI veto" model is provocative and elegant for an ed-tech DAO where Blue mediates strangers' funding requests. For a music + artist DAO where members are **chosen for taste**, an AI veto over an artist's proposal is a different bargain. ZAO's positioning advantage is: "ZOE assists, ZAO members decide." Borrow the **scoring rubric + DON signature pattern**, but invert the weight default (1% / 2% / 3% / 4% advisory vote vs. MWA's 10/20/30/40% structural vote).

## Build-on-Top Matrix (for The ZAO)

This is the action surface. Each row is shippable in 1-4 weeks and produces something demoable to James.

| # | ZAO Surface | Borrowed MWA Primitive | Concrete Deliverable | Effort | Sub-doc |
|---|---|---|---|---|---|
| 1 | **WaveWarZ pricing** | Black-Scholes binary + Quarter-Kelly + Avellaneda-Stoikov spreads from platform/lib/markets | Port `lib/markets/black-scholes.ts` + `lib/markets/kelly.ts` into WaveWarZ repo. Wire to existing Polymarket signal pipe. Add 3% edge threshold + 5%/40% caps as config. | M (1 week) | [647b](./647b-platform-treasury-trading.md) |
| 2 | **ZOE advisory scoring** | Blue 6-dim rubric (clarity/impact/feasibility/budget/ingenuity/chaos) + Eliza scoring contract | Build ZOE "Proposal Pre-Flight" skill: scores any ZAO proposal in 6 dims, returns advisory level 1-4 (NOT veto), posts as Farcaster reply. Optionally write CRE workflow later. | M (1-2 weeks) | [647a](./647a-platform-governance.md) |
| 3 | **ZAO Member onboarding** | survey-iq psychometric flow + Academic Angels NFT mint pattern | "ZAO Initiation" — Big Five + music-taste survey, mints a tiered ZID badge based on responses. Reuse Privy + Wagmi + OnchainKit wiring directly. | M (2 weeks) | [647e](./647e-privacy-data-tools.md) |
| 4 | **ZAO curriculum / Bootcamp** | EtherealHorizonPathway 14-seal SHA-256 system + 6 quest types | "ZAO Path" contract: N-milestone seal pathway for Bootcamp graduates, with proof-required + social + sealed-week quest types. Custom audio layer is portable. | L (3 weeks) | [647c](./647c-platform-frontend-pathway.md) |
| 5 | **WaveWarZ scenario sim** | Pocket-World multi-agent sim (Flask + Zep GraphRAG + OASIS) for doc→futures forecasting | "WaveWarZ Pre-Sim" — feed any open market into Pocket-World, get a 72-hour simulated discourse forecast, surface to predictors as one of many inputs. Run Pocket-World as separate Python service. | L (3-4 weeks) | [647d](./647d-pocket-world-azure.md) |
| 6 | **ZAO contributor payroll** | academy-crm Azura oracle CRE pattern (6-asset RWA, payroll automation) | Spec a "ZAO Payroll" CRE workflow: monthly USDC distribution to verified contributors based on Respect tokens. Sepolia first, mirror academy-crm's architecture. | L (3 weeks) | [647f](./647f-financial-governance.md) |
| 7 | **BCZ visitor analytics** | research repo's client-side stats workbench | Add a /bcz/visitors dashboard: client-side p-value tests on visitor cohort metrics. No server, no tracking pixel. | S (3 days) | [647e](./647e-privacy-data-tools.md) |
| 8 | **ZAO brand template** | brand-kit modular pattern (PDF + editorial-style-guide.md + Horizon_stone palette + horizonlanguage.json) | Mirror in zao-brand-kit: ZAO-style-guide.md + zao-palette.md + zaolanguage.json (vocabulary lockfile). One commit. | S (1 day) | [647g](./647g-peripheral-surfaces.md) |
| 9 | **ZAO content pipeline** | Halftone image processor (Next.js 14, hexagonal/square/triangular grids, PNG export) | Fork Halftone, retheme as `zao-halftone`. Generate every COC Concertz / WaveWarZ / FISHBOWLZ thumbnail through it. Unified visual language. | S (2 days) | [647g](./647g-peripheral-surfaces.md) |
| 10 | **ZAO inverse positioning** | MWA's EDITORIAL.md tone + Coursera/Gitcoin/ai16z positioning triangle | Write ZAO equivalent of EDITORIAL.md. Position vs Sound.xyz (extraction model), vs Royal (tokenization-first), vs Drakula (algo-first). State the inverse: "Artist-centric, member-curated, AI-respected." | S (1 day) | [647h](./647h-thesis-manifesto.md) |
| 11 | **MWA partnership proposal** | All of the above | Compose proposal to James: ZAO will run Bootcamp '26 cohort on EtherealHorizonPathway. Reverse: invite James as a Bootcamp guest. Ship a joint Farcaster miniapp. | S (1 day, after demos) | n/a |

## "Show James" Sequencing

The fastest path to a real conversation with James, in order:

1. **Day 1** — Ship row 10 (ZAO EDITORIAL.md inverse-positioning doc) and row 8 (zao-brand-kit). Post both publicly on Farcaster, tag @jinn. Cost: 1 day. Demonstrates Zaal read the brand stack carefully.
2. **Week 1** — Ship row 1 (port Black-Scholes + Kelly into WaveWarZ). Open a PR on platform repo with a comment: "stole this from y'all, here's a generalization that handles edge=0 and t<1hr cleanly." Cost: 1 week. Demonstrates code-level engagement, opens technical dialogue.
3. **Week 2** — Ship row 2 (ZOE 6-dim advisory scoring) and row 7 (BCZ visitor stats). Both visible on Farcaster within 14 days. Cost: 1-2 weeks.
4. **Week 3** — Cold DM James on Farcaster (@jinn) or via research@mentalwealthacademy.net. Subject: "Saw your 14-seal pathway. Built one for music. Want to compare notes?" Attach row 3 (ZAO Initiation survey-iq fork) + row 4 (ZAO Path contract design doc).
5. **Week 4+** — If James responds, propose row 11 (joint Bootcamp cohort).

**What NOT to do:** Don't lead with "let's merge our DAOs" — different theses, different communities, different governance defaults. Lead with code, not vision.

## Also See

- [56 — ORDAO Respect System](https://github.com/bettercallzaal/zaoos/tree/main/research/governance/056-ordao-respect-system) — counterpoint to Blue's AI veto
- [149 — BuilderOSS Deep Dive](https://github.com/bettercallzaal/zaoos/tree/main/research/governance/149-buildeross-deep-dive-everything)
- [234 — OpenClaw Comprehensive Guide](https://github.com/bettercallzaal/zaoos/tree/main/research/agents/234-openclaw-comprehensive-guide)
- [245 — ZOE Upgrade Autonomous Workflow](https://github.com/bettercallzaal/zaoos/tree/main/research/agents/245-zoe-upgrade-autonomous-workflow-2026) — host for Blue-style scoring
- [421 — WaveWarZ patterns](https://github.com/bettercallzaal/zaoos/tree/main/research/wavewarz/421-) — host for Black-Scholes + Kelly port

## Next Actions

| Action | Owner | Type | By When |
|--------|-------|------|---------|
| Promote this doc bundle (647 + 647a-h) from bcz-journal to bettercallzaal/zaoos canonical lib | @Zaal | PR | After review |
| Write ZAO inverse-EDITORIAL.md (row 10) | @Zaal | Repo commit | Day 1 |
| Mirror brand-kit pattern as zao-brand-kit (row 8) | @Zaal | New repo | Day 1 |
| Port Black-Scholes + Quarter-Kelly into WaveWarZ + open PR upstream on MWA platform (row 1) | @Zaal | PR | Week 1 |
| Build ZOE 6-dim advisory scoring skill (row 2) | @Zaal + ZOE | Skill | Week 1-2 |
| Fork survey-iq pattern for ZAO Initiation flow (row 3) | @Zaal | New surface | Week 2 |
| Design ZAO Path contract from EtherealHorizonPathway (row 4) | @Zaal | Solidity | Week 3 |
| Cold DM @jinn on Farcaster with proof-of-engagement (rows 1+2+7) | @Zaal | DM | End of Week 2 |
| Re-validate doc on 2026-06-13 (30-day SLA) | @Zaal | Review | 2026-06-13 |

## Sources

### Primary (MWA org)
- Org page: https://github.com/Mental-Wealth-Academy
- Org profile README: https://github.com/Mental-Wealth-Academy/.github/blob/main/profile/README.md
- platform repo: https://github.com/Mental-Wealth-Academy/platform
- platform README: https://github.com/Mental-Wealth-Academy/platform/blob/main/README.md
- platform EDITORIAL.md: https://github.com/Mental-Wealth-Academy/platform/blob/main/EDITORIAL.md
- Live governance contract: https://basescan.org/address/0x2cbb90a761ba64014b811be342b8ef01b471992d
- Academic Angels NFT: https://basescan.org/address/0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f

### Live deploys
- mentalwealthacademy.world (platform)
- mwa-treasury.vercel.app (treasury)
- mwa-genetics.vercel.app (genetics)
- mwa-research.vercel.app (research)
- azure-world.vercel.app (Pocket-World)
- academyos.vercel.app (wellness-directory)

### People
- James Marsh GitHub: https://github.com/jqmwa
- @jinn on Farcaster (1928 followers as of 2026-05-13)
- @MentalWealthDAO on X
- Contact: research@mentalwealthacademy.net

### Vendor refs
- Chainlink CRE docs: https://chain.link/cre
- ElizaOS: https://github.com/elizaOS/eliza
- milady upstream: https://github.com/milady-ai/milady
- Optimism RetroPGF: https://app.optimism.io/retropgf
