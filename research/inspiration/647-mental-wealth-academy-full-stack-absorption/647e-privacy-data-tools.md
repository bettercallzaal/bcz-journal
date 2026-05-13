---
topic: inspiration
type: comparison
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647e - Privacy-First Data Tools Cluster

## TL;DR

Mental Wealth Academy operates 3 complementary privacy-first tooling repos: genetics, a browser-native genomics lab that processes 110k+ SNPs via WebAssembly SQLite without server upload; research, a statistical workbench implementing exact p-values via beta function approximation and producing publication-ready box plots + correlation matrices; and survey-iq-initiation, a gamified psychometric onboarding system with blockchain membership verification (Academic Angels NFT on Base chain) and Farcaster identity integration. Zero exfiltration across all 3; all compute stays client-side or in minimal serverless verification handlers.

## Three Tools Compared

| Tool | Input | Compute Location | Output | On-Chain Hook |
|------|-------|------------------|--------|---------------|
| **genetics** | 23andMe/Ancestry/MyHeritage CSV; auto-format detection | Web Worker + sql.js (WASM) browser SQLite | Matched SNPs, genoset compounds, pharmacogenomics flags, magnitude scoring | None (data never written; privacy-preserving) |
| **research** | CSV paste/drag or manual schema-first entry; auto-type inference (80% threshold for numeric) | Client-side TypeScript stats engine; Lanczos gamma + Lentz beta CF | Descriptive stats, t-test/ANOVA/regression/correlation with exact p-values; scatter/histogram/bar/line/boxplot charts; correlation heatmap; AI narrative via on-demand Claude call | None (localStorage feedback only) |
| **survey-iq-initiation** | IQ quiz (5 questions: pattern, logic, sequence, analogy, Fibonacci); Farcaster auth | React SPA + Wagmi contract read/write; Neynar FID lookup | IQ score + feedback; member record with FID + verified address | Academic Angels NFT mint (0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f on Base); Farcaster profile write via Neynar SDK |

## genetics

**Repository**: https://github.com/Mental-Wealth-Academy/genetics  
**Live**: https://mwa-genetics.vercel.app  
**Size**: 353 KB; Created 2026-02-27; Updated 2026-03-01  
**Tech**: Next.js 14 (App Router), React 18, TypeScript 5.2, sql.js 1.14, Comlink 4.4, React Virtuoso 4.18, infobox-parser 3.6

### Architecture

All DNA processing runs client-side in the browser. No genetic data is sent to any server. The pipeline:

1. **File Upload & Format Detection** — User uploads genetic file (23andMe, Ancestry, MyHeritage, FamilyTreeDNA). Parser registry (infobox-parser) auto-detects format from file headers. Supported formats: .txt, .csv, .zip (auto-extracted).

2. **Main Thread Parsing** — Raw file content parsed into structured genotype objects. Column extraction and type coercion. Four parsers registered: parser23andMe, parserAncestry, parserMyHeritage, parserFTDNA (via /lib/genetics-parsers/index.ts).

3. **Web Worker SNP Matching** — The core workload delegates to a background thread via Comlink 4.4 to keep the browser UI responsive during 110k+ SNP queries. Worker wraps sql.js SQLite instance and executes match queries asynchronously.

4. **WebAssembly SQLite** — sql.js 1.14 compiles SQLite to WASM (public/sql-wasm.wasm, 1.2 MB compiled binary). In-memory relational database loaded with SNPedia snapshot. Queries executed in worker; results streamed back via Comlink proxy.

5. **Result Virtualization** — Matched SNP list rendered via React Virtuoso 4.18, which only mounts visible rows (low memory footprint for large result sets).

### SNP Matching Algorithm

Upon file upload, the parser extracts user's genotypes at each SNP position (e.g., rs1234567 = AA). The worker queries the local SQLite database for that rs-ID and compares user genotype to SNPedia genoset definitions. For each matched genotype:

- Extract **magnitude** (0-4 scale) indicating clinical relevance
- Extract **genotype-specific summary** (e.g., "increased caffeine metabolism")
- Flag **pharmacogenomics** drug-gene interactions (e.g., BRCA1 + breast cancer risk, CYP450 variants + warfarin metabolism)
- Store compound **genosets** (multi-SNP patterns e.g., high-magnitude APOE4/APOE4 + neuroinflammation risk)

No batch uploads. No server-side variant calling. Pure client-side pattern matching against a public reference.

### Pharmacogenomics Rules Engine

Hardcoded rules map genotypes to clinical phenotypes:

- **BRCA1/BRCA2 variants** — Breast/ovarian cancer hereditary risk
- **CYP450 polymorphisms** — Drug metabolism (warfarin, clopidogrel, codeine)
- **APOE4 alleles** — Alzheimer's disease risk
- **Factor V Leiden** — Thrombophilia risk

Each result includes magnitude + summary; no prescription recommendations (clinical counselor responsibility).

### Privacy Guarantees

- Zero network requests for genetic data
- SQLite queries execute in WASM sandbox (cannot leak to DOM/XHR)
- User can inspect DevTools Network tab; no hidden POST/PUT calls
- Session storage only (no persistence of uploaded files)

### Component Tree

```
app/genetics/page.tsx (main layout)
├── components/genetics/FileUpload (drag-drop)
├── components/genetics/GeneticsChat (Q&A, local rule-based responses)
├── components/genetics/GenosetDisplay (compound genoset results)
├── components/genetics/ResultsDisplay (matched SNP list + detail cards)
├── components/genetics/SNPBrowser (filtered/sorted SNP table)
├── components/genetics/WikiContent (SNPedia infobox rendering)
├── hooks/useSNPMatcherWorker (Comlink proxy to worker)
└── app/api/genetics/chat/route.ts (Node.js endpoint for AI chat, currently disabled)
```

### Chat Endpoint

Route: `/api/genetics/chat`  
Current implementation: Local rule-based responses (no LLM integration). Handler checks message keywords (e.g., "privacy", "BRCA", "genoset") and returns pre-written explanations. No Claude/OpenAI calls. Future path: integrate Claude API for open-ended genomics questions with SNP context injected.

---

## research

**Repository**: https://github.com/Mental-Wealth-Academy/research  
**Live**: https://mwa-research.vercel.app  
**Size**: 63 KB; Created 2026-02-27; Updated 2026-03-01  
**Tech**: Next.js 14, React 18, TypeScript 5.2, Chart.js 4.5, react-chartjs-2 5.3, react-markdown 10.1, pg 8.16 (stub)

### Statistical Methods Exposed

#### Descriptive Statistics (client-side)

For each numeric variable:

- **Mean** — Arithmetic average
- **Std Dev** — Bessel-corrected (n-1 denominator)
- **Median** — Midpoint of sorted array
- **Min/Max** — Range extrema
- **Q1/Q3** — Linear interpolation at positions 0.25(n-1) and 0.75(n-1)
- **Skewness** — Fisher-Pearson adjusted: n / [(n-1)(n-2)] * sum[(xi - mean) / sd]^3
- **Excess Kurtosis** — Fourth moment minus 3

Example output for variable score with n=15:  
`Mean: 82.67 | SD: 14.23 | Med: 85.00 | Min: 54.00 | Max: 95.00 | Q1: 61.50 | Q3: 90.50 | Skew: -0.41 | Kurt: -1.18`

#### Inferential Testing

Four parametric tests with exact p-values from continuous CDFs (no lookup tables):

1. **Welch's Independent Samples t-Test** — Unequal variance comparison via Welch-Satterthwaite df approximation. Reports: t-statistic, df, two-tailed p, Cohen's d (pooled SD).

2. **Pearson Product-Moment Correlation** — Bivariate association. Transforms r to t-statistic with df=n-2. Reports: r, r^2, t, two-tailed p.

3. **Ordinary Least Squares Regression** — Simple bivariate fit. beta1 = r * (sd_y / sd_x); beta0 = mean_y - beta1 * mean_x. Reports: intercept, slope, slope t, slope p, R^2, fitted equation.

4. **One-Way ANOVA** — Between-group vs. within-group variance partitioning. F = MS_between / MS_within. Reports: F-statistic, df_between, df_within, MS values, exact p.

#### P-Value Engine

The core computation is the **regularized incomplete beta function** I_x(a, b), evaluated via Lentz's continued fraction algorithm (Numerical Recipes, Ch. 6):

```
log-gamma: Lanczos approximation (g=7, 9-term series) with reflection formula for z < 0.5
continued fraction: Modified Lentz with epsilon=3e-14, max 200 iterations
symmetry: Auto-swap args if x > (a+1)/(a+b+2) for rapid convergence
```

Student's t CDF derived: P(T <= t) = 1 - 0.5 * I[ df/(df+t^2) ](df/2, 0.5)  
Fisher-Snedecor F CDF: P(F <= f) = I[ d1*f/(d1*f+d2) ](d1/2, d2/2)

All p-values displayed to 4 decimal places; values < 0.0001 shown as `< .0001`.

### Visualization Stack

Five chart types via Chart.js 4.5 + react-chartjs-2 5.3 + custom SVG:

| Chart | Use Case | Encoding |
|-------|----------|----------|
| **Scatter** | Bivariate numeric | Color-coded series by categorical grouping |
| **Histogram** | Univariate distribution | Configurable bins (2-30) |
| **Bar** | Group means | Categorical X, numeric Y (mean aggregation) |
| **Line** | Sequential / time-series | Cubic Bezier interpolation (tau=0.3), area fill |
| **Box Plot** | Group distribution | Custom SVG: whiskers (min/max), IQR box, median rule, per-group n label |

Box plot example (score by group):  
```
Group A (n=8):  Min=82, Q1=85.5, Med=88.5, Q3=91.5, Max=95
Group B (n=7):  Min=54, Q1=56.5, Med=60.0, Q3=65.0, Max=72
```

#### Correlation Matrix

Interactive Pearson r matrix for selected numeric variables. Chromatically encoded:
- Diagonal: r=1.00 (blue)
- Strong positive (r > 0.50): green
- Moderate positive (r > 0.20): light green
- Negative (r < -0.20): orange
- Weak (|r| <= 0.20): grey

Column inclusion toggled via checkboxes. Select All control provided.

### AI Interpretation Layer (Azura)

On-demand Claude narrative synthesized from dataset state. Activated explicitly via **Generate Analysis** button (no auto-computation). Summary includes:

- Dataset dimensionality (N, variable counts by type)
- Top-3 variable descriptives
- Strongest pairwise correlation
- Most recent test result
- Statistical power advisory (N < 30 caveat)

Feedback mechanism: thumbs-up / thumbs-down ratings persisted to localStorage keyed by deterministic hash of dataset. Enables per-dataset feedback continuity across sessions.

### Data Shapes

**Input**: CSV pasted or dragged. Auto-detection parses quoted fields, mixed delimiters, sparse entries via streaming character-level parser with quote-state tracking. 80% numeric threshold heuristic for type classification.

**Schema**: User can define columns a priori (name + type: 'numeric' | 'categorical'), then populate row-by-row with validation.

**Output**: Descriptive stats, test results, visualizations (JSON chart configs), correlation matrix (JSON r-values). All computation client-side; no server persistence.

### Component Tree

```
app/research/page.tsx (main workbench)
├── Data Ingestion (CSV paste, manual entry, schema builder)
├── Descriptive Stats Panel
├── Visualization Panel (5 chart types)
├── Inferential Testing (t-test, ANOVA, regression, correlation)
├── Correlation Matrix Interactive
├── Azura Interpretation (Claude call, optional)
└── Readings Sidebar (curated markdown articles)
```

### Database Stub

File: `lib/db.ts`  
Current implementation is a stub that logs "Database not configured in standalone mode" and returns empty array. Full Supabase/PostgreSQL integration lives in platform repo (not exported here). Research tool operates independently with zero backend.

---

## survey-iq-initiation

**Repository**: https://github.com/Mental-Wealth-Academy/survey-iq-initiation  
**Live**: https://azuraos.netlify.app/  
**Size**: 18.6 MB; Created 2025-08-28; Updated 2026-02-27; Stars: 2  
**Tech**: React 18, TypeScript 5.9, Vite 7.1, Express 5.1, TailwindCSS 3.4, Wagmi 2.16, Viem 2.36, Coinbase OnchainKit 0.38, Farcaster Auth Kit 0.8, Privy 2.23, Neynar 1.2, Radix UI

### Psychometric Instruments

**IQ Quiz Component** (5 questions):

1. **Pattern Recognition** — Sequence problem: 2, 6, 12, 20, 30, ? (Answer: 42; differences +4, +6, +8, +10)
2. **Logical Reasoning** — Set inclusion: "If all Zeps are Blons and some Blons are Trels, which is true?" (Answer: Some Zeps are Trels)
3. **Sequence Completion** — ACE, BDF, CEG, DFH, ? (Answer: EGI; character incrementation)
4. **Fibonacci Sequence** — 1, 1, 2, 3, 5, 8, ?, ? (Answer: 13, 21)
5. **Analogies** — Book is to Reading as Fork is to ? (Answer: Eating)

Scoring:
- 0/5: Basic pattern failure feedback
- 1/5: Transitive logic struggles
- 2/5: Improved reasoning
- 3/5: Solid pattern matching
- 4/5: Strong analytical thinking
- 5/5: Exceptional cognitive performance

(IQTestModal.tsx hardcodes 6-level feedback scale; feedback references stored as record keyed by score.)

### Onboarding Flow

1. **Auth Gate** — User connects wallet (Privy, Coinbase OnchainKit, or standard Wagmi wallets; Farcaster via Auth Kit)
2. **IQ Quiz Modal** — 5-question assessment; 60-second time ceiling (configurable)
3. **Membership Modal** — Mint Academic Angels NFT (0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f on Base chain)
   - OnchainKit mintDetails hook pulls contract state
   - Quantity selector (1-5 default)
   - Scatter Art API integration (metadata fetch: https://api.scatter.art/v1)
   - Wagmi writeContract + waitForTransactionReceipt (waits for 1 confirmation)
4. **Friends Leaderboard** — Hardcoded top 5 AzuraOS token holders (FID + tokenBalance in lamports):
   - jamesdesign.eth (FID 286924): 13.5M tokens
   - jellofish (FID 1077966): 4.68M tokens
   - brennuet (FID 284618): 4.68M tokens
   - roadu (FID 194372): 2.34M tokens
   - jesse.base.eth (FID 99): 1.17M tokens
   - Neynar API enriches with username, display_name, pfp_url, verified_addresses (eth)

### Scoring Pipeline

Flow:
1. User takes quiz; responses collected
2. Score calculated (number of correct answers / 5)
3. Feedback string looked up from IQ_FEEDBACKS record
4. Membership Modal rendered (pass score as prop or store in React context)
5. Upon NFT mint confirmation, user record created with FID + score + tx hash

**On-Chain Write**: 
- Contract: Academic Angels (0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f, Base chain)
- Function: `mint(quantity: uint256)` (payable)
- Via Wagmi + Viem v2.36 for transaction serialization and signing
- Awaits transaction receipt before UI success state

### AI/LLM Integration

**Azura Terminal Modal** (currently disabled; returns null)  
File: `client/components/AzuraTerminalModal.tsx`  
Designed for on-demand AI governance persona but implementation stubbed. Original design:
- Terminal-like UI for conversational interactions
- RAG layer (vector embeddings for business/community docs in Supabase)
- Decision-making companion for DAO proposals
- Future hook for Azura AI persona on-chain voting

### Blockchain Integration

**Chains**: Base (primary); Ethereum compatible via Wagmi
**Identity**: 
- Farcaster Auth Kit (FID-based identity)
- Privy (social login fallback)
- Coinbase OnchainKit (native wallet UX)

**Contracts**:
- Academic Angels NFT: 0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f (ERC721 or ERC1155; mint function)
- Verified address extraction: user?.verified_addresses?.eth_addresses?.[0]

**Leaderboard Data Flow**:
1. Hardcoded FID list (TOP_AZURAOS_HOLDERS)
2. Neynar API bulk fetch (X-API-KEY header required) -> fetch user profiles
3. Map FID to username, display_name, pfp_url, eth address
4. Append token balance (hardcoded in record)
5. Return /api/friends-leaderboard (GET, CORS enabled)

### Component Tree

```
client/App.tsx (SPA entry point, React Router 6)
├── DaemonMenu (nav to game modes)
├── AzuraTerminalModal (AI companion, currently disabled)
├── IQTestModal (5-question quiz)
├── MembershipModal
│   ├── useMintDetails hook (OnchainKit)
│   ├── useAccount + useSwitchChain (Wagmi)
│   └── readContract + writeContract (Wagmi actions)
├── FriendsLeaderboard (hardcoded + Neynar-enriched)
├── RetroMusicPlayer (immersion)
└── UI Component Library (50+ Radix UI atoms + custom CSS)
```

### API Endpoints (Netlify Serverless)

- **GET /api/ping** — Health check (PING_MESSAGE env var)
- **POST /api/follow-user** — Neynar follow (stub; requires signer in future)
- **GET /api/friends-leaderboard** — Top holders + profile enrichment

### Environment

```
VITE_PUBLIC_BUILDER_KEY=<Builder.io API key>
PING_MESSAGE="ping pong"
VITE_CHAT_WEBHOOK_URL=https://jogibay.app.n8n.cloud/webhook/10207fda-f103-425e-9ab5-59950b3f5f9d/chat
VITE_API_URL=/api/chat
NEYNAR_API_KEY=5B907AF2-3087-49A8-88C6-DFA45CF19383
NEYNAR_CLIENT_ID=5B907AF2-3087-49A8-88C6-DFA45CF19383
ALCHEMY_RPC_URL=https://base-mainnet.g.alchemy.com/v2/F9OVxnYvtKwICSXHAbDvj
```

---

## Build-on-Top Hooks (for ZAO)

| ZAO Surface | Tool to Fork/Integrate | Use Case | Effort | Notes |
|------------|----------------------|----------|--------|-------|
| **Member Onboarding Flow** | survey-iq-initiation | Psychometric assessment for reputation scores; ZID credential binding | 2-3 weeks | Swap Academic Angels NFT for ZID genesis mint; attach IQ score to ZID record; gate reputation pipe-in on >50th percentile |
| **Reputation/Credential Engine** | genetics + research | Health-longevity-linked credentials; publish research + share SNP insights (with privacy consent) | 4-6 weeks | Genetics SNP pattern matching -> ZID credential (e.g., "longevity outlier" if APOE4-negative + high cardio SNPs); research publishing -> peer review rep bump; zero-knowledge proofs for genomic claims |
| **Privacy-First Analytics** | research | Visitor analytics for ZAO platform without Google Tag Manager; p-values on A/B tests | 2 weeks | Port chart.js + stats engine into ZAO dashboard; integrate session event stream; produce weekly stat summaries |
| **Data Custody Pattern** | genetics | Exemplar for ZAO client-side data processing (DeFi positions, wallet balances, etc.) | 1-2 weeks | Extract sql.js + Comlink patterns; generalize to ZAO treasury balance calculator; ensure zero-exfiltration in Vercel static export |
| **Gamified Governance** | survey-iq-initiation | ZAO DAO proposal voting as quiz-like quests (inspired by Azura Daemon Model narrative wrapper) | 3 weeks | Gamify proposal review; score understanding; lock-in participation badges; wire leaderboard to ZID reputation |

---

## Sources

- [Mental Wealth Academy Organization](https://github.com/Mental-Wealth-Academy)
- [genetics Repo](https://github.com/Mental-Wealth-Academy/genetics) — mwa-genetics.vercel.app
- [research Repo](https://github.com/Mental-Wealth-Academy/research) — mwa-research.vercel.app
- [survey-iq-initiation Repo](https://github.com/Mental-Wealth-Academy/survey-iq-initiation) — azuraos.netlify.app
- [Academic Angels NFT Contract](https://base.etherscan.io/address/0x39f259b58a9ab02d42bc3df5836ba7fc76a8880f) (Base chain)
- [Neynar Farcaster API](https://docs.neynar.com/)
- [OnchainKit Documentation](https://onchainkit.xyz/)
- [Wagmi Hooks](https://wagmi.sh/)
