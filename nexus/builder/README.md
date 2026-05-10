# For builders

You write code. You probably build with Next.js + Supabase + something onchain. You're skeptical of "web3 community" projects because most of them are vibes wrapped around a token. This page is the case for spending an hour on this one.

## What's actually here

A working, opinionated stack for **gated music communities on Farcaster**. It's been running for over a year, has 188+ on-chain Respect holders, and ships features weekly. Public, MIT-licensed, fork-friendly:

| Layer | Built |
|---|---|
| Auth | SIWF (Sign In With Farcaster) + SIWE + iron-session |
| Gating | Wallet allowlist, Farcaster allowlist, Respect-tier gating |
| Social | Farcaster channel feed, threaded replies, reactions, scheduled casts, search |
| DMs | XMTP (encrypted, MLS protocol) |
| Music | 9-platform inline player, crossfade, MediaSession, lock-screen controls, persistent queue, listening rooms, lyrics, waveform comments |
| Governance | Snapshot polls + Nouns Builder DAO + community proposals + Hats Protocol roles |
| Respect | OG (ERC-20 soulbound) + ZOR (ERC-1155) on Optimism, peer-evaluated via the Respect Game |
| Cross-posting | Farcaster + X + Bluesky + Hive (with normalization per platform) |
| Spaces | Stream.io live audio/video rooms, RTMP multistream, transcription, screen share |

The whole thing is documented in `community.config.ts` — change one file, you have your own community hub.

## Three things you can do, in order

### 1. Read the research before the code

I've been keeping a research library in [zaoos/research/](https://github.com/bettercallzaal/zaoos/tree/main/research) — **202 active docs** across music, governance, agents, infrastructure, cross-platform publishing, Farcaster Mini Apps. ~500K words, written while building. It's the part of the project that's most useful even if you never touch the codebase.

Two suggested entry points:

- **[research/community/050-the-zao-complete-guide](https://github.com/bettercallzaal/zaoos/tree/main/research/community/050-the-zao-complete-guide)** — what The ZAO is, end to end
- **[research/dev-workflows/154-skills-commands-master-reference](https://github.com/bettercallzaal/zaoos/tree/main/research/dev-workflows/154-skills-commands-master-reference)** — the dev workflow + skills setup that makes this pace possible

### 2. Fork ZAOOS

```
git clone https://github.com/bettercallzaal/zaoos.git
cd zaoos
npm install
cp .env.example .env.local       # fill in env vars
npm run dev                      # localhost:3000
```

Read [FORK.md](https://github.com/bettercallzaal/zaoos/blob/main/FORK.md) for the full setup (database scripts, app wallet, deploy). It's designed for you to run it locally in under an hour.

The single file you'll want to edit is [`community.config.ts`](https://github.com/bettercallzaal/zaoos/blob/main/community.config.ts) — branding, channels, contracts, admin FIDs, nav. Everything community-specific lives there.

### 3. Hire the connector

If you have a project where any of this is relevant — gated communities, music infra, on-chain reputation, Farcaster mini-apps, agent workflows — book time. I do paid consulting under "the Connector Treatment": problem-solving introductions, infrastructure builds, event production.

→ [calendly.com/zaalp99/30minmeeting](https://calendly.com/zaalp99/30minmeeting)

## Why this might be interesting

- **It's a real codebase, not a tutorial.** ~4200+ files, 100+ branches, tested in production. The graduation pattern (ZAOOS as lab → standalone repos as products) is one of the cleanest workflows I've seen for shipping multiple connected products from one codebase.
- **The version-stack pattern is documented.** I do architecture search by spinning new repos, not new branches. `NEXUSV1` → `NEXUSV5.8.5` was 16 separate repos in 21 days that eventually became ZAOOS. If you're skeptical, [look at the lineage](https://github.com/bettercallzaal/bcz-journal/tree/main/projects#nexus-lineage-history) — every one is still up.
- **The research stays public even when the code graduates.** Apps leave ZAOOS to live on their own, but the research that produced them stays in `zaoos/research/`. So the code might churn but the institutional memory doesn't.

## What this is NOT

- It's not generic. It's opinionated about Farcaster + Base/Optimism + Supabase. If you want a different stack, fork won't be useful.
- It's not stable. Active development, not a maintained product release.
- It's not free labor. I'm not going to debug your fork unsolicited. Calendly link if you want help.

---

Back to [the Nexus](../) · [Project index](../../projects/) · [Research mirror](../../research/) · [Calendly](https://calendly.com/zaalp99/30minmeeting)
