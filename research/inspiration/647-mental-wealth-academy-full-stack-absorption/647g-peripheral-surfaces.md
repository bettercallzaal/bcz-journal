---
topic: inspiration
type: guide
status: research-complete
last-validated: 2026-05-13
parent-doc: 647
tier: STANDARD
---

# 647g - Peripheral Surfaces

## TL;DR

Five satellite repos in Mental-Wealth-Academy org enable knowledge aggregation, wellness provider search, visual branding, image-to-halftone conversion, and iOS native app access. The Milady fork (upstream from milady-ai/milady) represents adoption of a privacy-first local-first AI agent architecture. Together, these create multi-channel, multi-platform access points to the core platform.

## Inventory

| Repo | Status | Purpose | Stack | Live URL | Last Push |
|------|--------|---------|-------|----------|-----------|
| knowledge | Active | Central hub: resource aggregation, framework synthesis, infrastructure docs | Markdown + GitHub | None (docs repo) | 2026-02-27 |
| wellness-directory | Active | Provider search surface + API | TypeScript + React | https://academyos.vercel.app | 2025-12-13 |
| brand-kit | Active | Brand guide + asset inventory | Markdown + PDF + JSON | https://mentalwealthacademy.world | 2026-04-27 |
| Halftone | Active | Halftone image generator tool | TypeScript + Next.js 14 | Not deployed (dev tool) | 2026-03-22 |
| mwa-swift | Early | iOS native app | Swift 5.9 + SwiftUI | None (early stage) | 2026-04-16 |
| milady (fork in org) | Active | AI agent runtime + chat | TypeScript/Node.js + ElizaOS | https://milady.ai | 2026-04-23 |

## knowledge

Central documentation hub for Mental Wealth Academy infrastructure and frameworks.

### Description & Purpose

"Central hub for aggregating, processing, and synthesizing knowledge — automated workflows, LLM-powered insights, and multi-channel dissemination."

Stores institutional knowledge in 4 top-level directories:

1. **Resources** - Curated protocols, toolkits, reference material for wellness programs; stress-response frameworks, neuroscience of interventions
2. **Infrastructure** - Shared systems documentation: treasury mechanics, governance flows, privacy architecture, autonomous agent operation
3. **Research** - Primary data, statistical analysis, sourced claims; "every framework the Academy publishes traces back to data"
4. **Frameworks** - Mental models and decision systems; cognitive architecture and governance operating system

### Live Architecture Details

Key systems documented in knowledge/:

- **Genomics Lab**: Browser-native, WebAssembly-based; 155MB SNPedia reference (110,000+ SNPs); processes 23andMe, AncestryDNA, MyHeritage, FamilyTreeDNA formats; no server-side storage of genetic data
- **Governance System**: On-chain (Base Mainnet); confidence levels 0-4 determine treasury exposure (10%-40%) and community threshold requirements (40%-10%)
- **Treasury**: Community-owned USDC pool on Base; autonomous trading via Black-Scholes, Avellaneda-Stoikov, Kelly criterion (0.25x fraction); market coverage: BTC, ETH, SOL, XRP, GOLD with 5-minute feeds; 3% edge threshold
- **Quest System**: Shard-based rewards; social, educational, wellness quest tracks; event reservation system
- **Daily Readings**: 3 curated pieces per day on DAO governance, AI agents, cyberpsychology, neuroscience
- **Chapter System**: Interactive long-form books; current: "Return to God" (3 chapters unlocked progressively)

### Tech Stack

No single language - documentation repo. Markdown files with embedded technical specs, JSON configs, API references.

### Repo Structure

```
knowledge/
├── README.md (core overview + systems)
├── frameworks/ (decision models, governance)
├── infrastructure/ (treasury, governance, privacy)
├── research/ (data, analysis, claims)
└── resources/ (protocols, toolkits, wellness)
```

### File Paths of Interest

- `/README.md` - 15KB+ comprehensive system overview
- `/infrastructure/` - Governance proposal confidence tables, treasury trading parameters
- `/resources/` - Neuroscience frameworks, stress-response toolkits
- `/frameworks/` - Cognitive models, decision architectures

### Relation to Other MWA

- **Upstream for**: wellness-directory (provider database), platform (all frameworks), academy-crm (CRE workflows), genetics (genomics research sourcing)
- **Input from**: genetics lab research, treasury trades, governance proposals (feeds daily readings)

---

## wellness-directory

Provider search surface for mental wellness and education professionals.

### Description & Purpose

"Wellness Provider Search" — Frontend interface for discovering, filtering, and connecting with wellness providers integrated into the Academy ecosystem.

Deployed live at: https://academyos.vercel.app (HTTP 200, active Vercel deployment)

### Tech Stack

- **Language**: TypeScript
- **Frontend**: React 18 + Vite (SPA) + TailwindCSS 3
- **Backend**: Express.js + Node.js
- **Auth**: Privy (email, social, wallet)
- **UI**: Radix UI + shadcn/ui
- **State**: React Context API
- **Routing**: React Router 6
- **AI**: Coinbase AgentKit for provider lookup + blockchain balance queries
- **Network**: Base (Ethereum L2)
- **Build**: Vite (client) + Vite (server config) + Vercel deployment

### Key Features (from package.json analysis)

- **Real-time blockchain balance lookups** using viem for provider accounts
- **Provider database with onchain account info** (wallet addresses, upcoming events)
- **Event reservation system** (dates, times, details)
- **Full Radix UI component palette** (30+ components: accordion, dialog, tabs, slider, etc.)
- **Analytics**: Vercel Analytics integration
- **Three.js**: React Three Fiber + Drei for 3D visualization
- **Query client**: TanStack React Query for data fetching

### Build Process

```bash
npm run build:client       # Vite build to dist/spa
npm run build:server       # Vite server config to dist/server
npm run copy:well-known    # Copy .well-known/ to dist/spa
npm start                  # Node dist/server/node-build.mjs
```

### Live Deployment

- **URL**: https://academyos.vercel.app
- **Status**: 200 OK, live
- **Platform**: Vercel
- **Domain**: academyos (Academy Operating System - refers to entire platform)

### Relation to Other MWA

- **Consumes**: knowledge/ (provider frameworks), genetics (optional wellness genetic insights)
- **Feeds**: platform (provider events, user provider selections)
- **Powers**: wellness quest completions (Academy members can reserve provider sessions)

---

## brand-kit

Brand assets, design system, and editorial guidance for Mental Wealth Academy.

### Description & Purpose

"Brand guide and logos for Mental Wealth Academy. Next Gen Learning."

Single source of truth for visual identity, typography, color system, and editorial voice.

### Contents

Top level: README.md, funding.json, guides/

**guides/** inventory:
- `brand-guide.pdf` (2.1 MB) - Visual identity documentation (fonts, colors, logo usage)
- `editorial-style-guide.md` - Writing voice, tone, structure guidelines
- `Horizon_stone.md` - Color palette reference (Horizon Stone design system)
- `horizonlanguage.json` (4.1 KB) - Language tags, terminology mapping, possibly i18n config

### Design System: Horizon Stone

From filename and structure, MWA uses a custom design language called "Horizon Stone":
- Color palette defined in horizonlanguage.json (likely hex values, name mappings)
- Typography rules (fonts specified in brand-guide.pdf)
- Logo and usage guidelines

### Topics (from GitHub API)

- ai-agents
- education
- governance

### Live Presence

- **Primary URL**: https://mentalwealthacademy.world (HTTP 307 redirect to https://www.mentalwealthacademy.world)
- **Status**: Active, resolving brand page
- **Purpose**: Public-facing brand + asset hosting

### Repo Details

- **Type**: Documentation/design assets
- **Primary Language**: None (PDF + Markdown + JSON)
- **Last Updated**: 2026-04-27
- **Typically**: Consumed by platform, wellness-directory, Halftone (design consistency)

### File Paths

- `/guides/brand-guide.pdf` - Master visual identity (2.1 MB)
- `/guides/editorial-style-guide.md` - Voice and tone
- `/guides/Horizon_stone.md` - Color/stone definitions
- `/guides/horizonlanguage.json` - Terminology config (4.1 KB)
- `/README.md` - Brand kit overview + usage

### Relation to Other MWA

- **Consumed by**: platform (UI theming), wellness-directory (provider UI), Halftone (image styling), mwa-swift (iOS design)
- **Referenced in**: milady (customization for MWA fork)

---

## Halftone (Mystery Solved)

Next.js-based halftone image processor tool with artistic/visual effects.

### Discovery

Initial probe: No README, no description on GitHub. Investigation revealed purpose from commit messages and code analysis.

### What It Is

**Halftone Generator**: A web application that transforms images into halftone prints using adjustable grids, dot styles, and preprocessing filters.

From initial commit (2026-03-22):

```
Initial Halftone generator app

Next.js 14 halftone image processor with MWA design system.
Supports hexagonal/square/triangular grids, circle/square/diamond/line
dot styles, gamma/contrast/blur preprocessing, and PNG export.
```

### Tech Stack

- **Language**: TypeScript
- **Framework**: Next.js 14.2.0
- **Frontend**: React 18.3.0 + React DOM 18.3.0
- **Styling**: Next.js built-in CSS modules
- **Dev Tools**: TypeScript 5.4.0, @types/react, @types/node

### App Structure

```
app/
├── page.tsx (main component - 80+ lines)
├── page.module.css
├── layout.tsx
└── globals.css

components/
├── Navbar (MWA branding)
├── Inspector (parameter controls)
└── HalftoneCanvas (rendering engine)
```

### Feature Set

**HalftoneParams interface** (from page.tsx):

Grid types: hexagonal, square, triangular
Dot styles: circle, square, diamond, line
Preprocessing: blur, gamma, contrast
Channels: luminance, red, green, blue
Advanced: grid rotation, base resolution (1420px default), spacing (6px default)
Min/max output mapping (0-1 range default)

Default params:
- Grid: hexagonal, 6px spacing, -143 degrees rotation
- Blur: 8
- Gamma: 0.70
- Contrast: 7
- Channel: luminance
- Dot style: circle

**Interaction**:
- Real-time param adjustment
- Undo/redo history stack
- PNG export capability
- Render time tracking
- Image stats display (width, height, dot count)

### UX Flow

1. Upload or select image
2. Adjust halftone parameters via Inspector
3. HalftoneCanvas renders live preview
4. Export PNG with current settings
5. Undo/redo parameter history

### Deployment

**Not deployed** to live URL. Dev tool maintained in repo.

Most recent commit (2026-03-22): "Replace navbar logo with MWA branding, make navbar taller"

Indicates: Tool integrated into MWA visual workflow, used internally for brand asset generation or educational content.

### Purpose in MWA Ecosystem

**Best guess**: Used to generate artistic halftone versions of course materials, educational content, or brand assets for visual impact. Could be:
- Content authoring tool for daily readings
- Visual effect filter for chapter illustrations
- Brand asset generation for social media
- Educational demonstration of image processing techniques

### Relation to Other MWA

- **Consumes**: brand-kit (Horizon Stone color system, navbar styling)
- **Feeds**: platform (educational visual content), possibly generates assets for research/

---

## mwa-swift

Native iOS application for Mental Wealth Academy.

### Discovery

Initial: No README, no description. Investigation reveals from code and Package.swift.

### What It Is

**Native iOS App** for Mental Wealth Academy, providing mobile access to core platform features.

From initial commit (2026-04-16):

```
Initial Swift/SwiftUI iOS app

Native iOS app for Mental Wealth Academy - Course, Companion, Account tabs.
Calls existing Vercel API endpoints. Auth via Privy iOS SDK.
```

### Tech Stack

- **Language**: Swift 5.9
- **Framework**: SwiftUI (native iOS UI)
- **Minimum iOS**: iOS 17+
- **Key Dependencies**:
  - Privy iOS SDK (1.0.0+) - Wallet auth, email/social login
  - XCTest (implicit) - Testing framework
- **Project Management**: Swift Package Manager + Xcode project
- **Assets**: Fonts folder + xcassets (images)

### App Structure

```
Sources/
├── App/
│   ├── MentalWealthAcademyApp.swift (app entry point)
│   ├── Assets.xcassets/ (images, icons)
│   └── Fonts/
├── Account/
├── Auth/
├── Companion/
├── Course/
├── Navigation/
├── Paywall/
└── Shared/
```

### Feature Tabs (3-tab layout)

1. **Course** - Educational content consumption
2. **Companion** - AI companion chat (likely Azura integration)
3. **Account** - User profile, wallet, settings

Plus: Auth module (Privy), Navigation (tab switching), Paywall (subscription/events), Shared (UI components)

### API Integration

Calls existing Vercel API endpoints from wellness-directory + platform (academyos.vercel.app).

Authentication via Privy iOS SDK - supports email, social, wallet connection.

### Recent Activity

3 commits (all on 2026-04-16):
1. "Fix Privy SDK product name: PrivySDK → Privy"
2. "Add Assets.xcassets and Fonts folder"
3. "Initial Swift/SwiftUI iOS app" (initial commit)

Indicates: Early stage, recently scaffolded, still integrating Privy SDK.

### Deployment / Distribution

**Not yet live**. No App Store link provided. Code-stage development.

Expected distribution: App Store + TestFlight when ready.

### Relation to Other MWA

- **Consumes**: platform API (https://academyos.vercel.app) for courses, companion, account data
- **Auth**: Privy iOS SDK (same auth system as web platform)
- **Design**: brand-kit (Horizon Stone design language for iOS)
- **Future**: Could subscribe to knowledge updates, wellness provider searches, quest tracking

---

## milady (Fork in Org - NOT an Original Fork)

The milady repo in Mental-Wealth-Academy is **not a fork created within the org**, but rather the **upstream milady-ai/milady repository itself**, indicating MWA adopted the architecture as their AI agent runtime.

### What Is Milady

**Privacy-first AI agent framework** and local-first assistant framework.

Official description: "your schizo AI waifu that actually respects your privacy" (terminally online vibe)

Built on elizaOS (open-source agent framework), provides:
- Local-first runtime (agents run client-side first)
- Optional Eliza Cloud connection for hosted runtime
- Multi-platform: Desktop (macOS, Windows, Linux), iOS, Android
- Gateway control plane for session/tool/vibe management
- Telegram, Discord, WebChat UI integrations

### Tech Stack

- **Language**: TypeScript + Node.js
- **Core Deps**: elizaOS (ElizaOS agent framework), AgentKit (Coinbase), Viem (blockchain), Wagmi (web3)
- **Workspace**: Monorepo (apps/, eliza/, plugins/)
- **Runtime**: v22 Node.js minimum
- **Version**: 2.0.0-alpha.116 (active development)
- **Package Manager**: Bun (package audits with bun audit)

### MWA Adoption

MWA Mental-Wealth-Academy/milady is a **copy/fork of the upstream repository** indicating:

1. **Privacy-first philosophy alignment**: Both MWA and Milady prioritize local-first, privacy-respecting architecture
2. **AI agent customization**: Likely customizing Milady for MWA's governance proposals, treasury interactions, wellness recommendations
3. **Multi-platform access**: Milady's iOS/Desktop app infrastructure supports MWA's desire for native app access (aligns with mwa-swift effort)

### Features Leveraged by MWA

**BNB/BSC Integration**: Milady ships with native BNB Smart Chain support - agents can trade tokens, track launches, interact with DeFi. Relevant if MWA extends beyond Base to multi-chain.

**Trading**: Built-in EXECUTE_TRADE action for token swaps via PancakeSwap (if configured).

**Meme Token Discovery**: Meme Rush integration (real-time token tracking).

**Wallet**: Auto-generates EVM and Solana wallets. Supports Privy for managed wallets.

**Desktop App**: Multi-platform native apps (macOS arm64/x64, Windows, Linux, iOS, Android).

**Telegram/Discord Integration**: Agents connect to messaging platforms - could extend MWA community engagement.

### Recent Activity (MWA Fork)

3 commits (mostly bots, last human: 2026-04-23):
- 2026-04-23: "Format bootstrap script" (bot)
- 2026-04-23: "Merge remote-tracking branch 'origin/develop'" (bot)
- 2026-04-23: "Update eliza LifeOps dashboard" (bot)

Indicates: MWA is tracking upstream milady development, auto-syncing updates.

### Source & Upstream

- **Upstream**: github.com/milady-ai/milady
- **Fork Date**: Feb 2026 (milady-ai org created 2026-02-07)
- **MWA Copy Date**: April 2026 (milady repo appears on 2026-04-23)
- **Active Syncing**: Yes - auto-bot commits pulling from develop branch

### Homepage & Resources

- **Official Milady Site**: https://milady.ai (200 OK)
- **Desktop Downloads**: Releases page (macOS DMG, Windows EXE, Linux AppImage/deb)
- **Mobile**: App Store (iOS), Google Play (Android)

### Relation to Other MWA

- **Integrates with**: platform (Azura could leverage Milady runtime for enhanced agent capabilities)
- **Supports**: mwa-swift philosophy (privacy-first, local agent, mobile-native)
- **Feeds**: wellness-directory (agent-assisted provider search), knowledge (daemon swarm research agents)
- **Aligns with**: Governance proposal review (Azura confidence scoring could use Milady's multi-turn reasoning)

---

## Build-on-Top Hooks (For ZAO)

### Knowledge Aggregation Patterns

**From knowledge/ repo:**

MWA centralizes institutional knowledge in structured directories (resources, infrastructure, research, frameworks). ZAO can adopt the same pattern:

1. Create `/knowledge/ZAO-resources/` with wellness/governance protocols
2. Link to `/infrastructure/` for ZAO treasury mechanics (reuse MWA patterns)
3. Feed daily readings from curated ZAO domain experts
4. Generate frameworks for ZAO decision systems

**Reusable artifact**: The knowledge README itself is a master template - it documents the full system architecture in prose, making it fork-friendly.

### Wellness Directory Schema & Provider Search

**From wellness-directory:**

The provider database schema (onchain account info, event scheduling, profile data) is a portable pattern. ZAO can:

1. Fork wellness-directory package.json structure (Vite + Vercel stack)
2. Point to ZAO provider network instead of MWA
3. Reuse AgentKit provider lookup (blockchain balance queries)
4. Adapt Radix UI theming for ZAO brand-kit

**Build angle**: Multi-tenant provider directory system - wellness-directory becomes a template for communities building local provider networks.

### Brand-Kit as Fork-Friendly Template

**From brand-kit:**

MWA's structure (brand-guide.pdf + editorial-style-guide.md + Horizon_stone.json + horizonlanguage.json) is a portable brand system.

ZAO could:

1. Copy brand-kit structure exactly
2. Replace Horizon Stone palette with ZAO color system
3. Swap editorial voice guidelines for ZAO tone (if different)
4. Publish JSON terminology mappings (horizonlanguage.json) for i18n + consistency

**Reuse pattern**: Brand kits are often one-off docs. Structuring it as a modular repo (separate PDFs, JSON configs, markdown guides) makes forking trivial.

### Halftone as Content Generation Tool

**From Halftone:**

Image-to-halftone processing could be repurposed:

1. ZAO daily readings - halftone filter for visual impact on research content
2. Quest asset generation - convert course illustrations to stylized prints
3. Community-created content - allow members to halftone their submissions

**Reuse**: Halftone is a dev tool, not a consumer app, making it easy to integrate into content pipelines.

### Privacy-First Mobile Strategy (mwa-swift + Milady)

**From mwa-swift + Milady:**

MWA invests in 2 agent approaches:

1. **Swift native app** (mwa-swift) - Direct iOS control, offline-capable
2. **Milady runtime** - Local-first agent with optional cloud sync

ZAO could adopt both:

1. **iOS app from mwa-swift**: Copy Swift package structure, point Privy SDK to ZAO auth
2. **Local agent runtime**: Embed Milady into ZAO for privacy-first research agents, community governance bots, wellness tracking without cloud dependency

**Pattern**: Privacy-first is a build-on-top feature, not just architecture - brand it, make it a thesis point.

---

## Sources

- https://github.com/Mental-Wealth-Academy/knowledge (README: central hub architecture)
- https://github.com/Mental-Wealth-Academy/wellness-directory (Vercel deploy, provider search)
- https://github.com/Mental-Wealth-Academy/brand-kit (Horizon Stone design system)
- https://github.com/Mental-Wealth-Academy/Halftone (Next.js halftone image tool)
- https://github.com/Mental-Wealth-Academy/mwa-swift (iOS native app, Swift 5.9)
- https://github.com/Mental-Wealth-Academy/milady (fork of milady-ai/milady, privacy-first AI agent runtime)
- https://academyos.vercel.app (live wellness-directory deployment, HTTP 200)
- https://mentalwealthacademy.world (brand-kit homepage, HTTP 307 to www subdomain)
- https://milady.ai (Milady AI official site, privacy-first agent framework)
- https://github.com/milady-ai/milady (upstream: Milady framework origin)

