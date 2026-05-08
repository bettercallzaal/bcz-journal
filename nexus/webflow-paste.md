# Webflow Nexus — paste-ready copy

> **This file is a paste source for `thezao.com/nexus`. It is NOT meant to be the final live page on its own — it's the content blocks ready to drop into Webflow.**
>
> Each section below is a self-contained block. Drop them into Webflow as separate sections with hero / split / 3-up / link grid templates.

---

## SECTION 1 — Hero

**Headline:** ZAO Nexus
**Subhead:** One door into everything. The ZAO is an impact network for independent musicians on Farcaster + Base. WaveWarZ is the onchain music battle product. ZABAL is the streaming layer that ties them together.
**Primary CTA:** Join the ZAO → /zao-token (or current join flow)
**Secondary CTA:** Read the journal → https://bettercallzaal.github.io/bcz-journal/

---

## SECTION 2 — The pillars (4-up grid)

### Card 1: The ZAO
**Eyebrow:** The impact network
**Body:** Decentralized hub for independent musicians, artists, and technologists. Members earn soulbound Respect for showing up. Build to belong.
**Link:** Explore → /community

### Card 2: WaveWarZ
**Eyebrow:** Onchain battles
**Body:** Solana-native music battles. Live every weeknight at 8:30 PM EST. 906 battles, $42K+ traded. Real artist payouts on every win.
**Link:** Watch live → https://www.wavewarz.com

### Card 3: ZABAL
**Eyebrow:** The new front-end
**Body:** Streaming + coordination layer for the whole ecosystem. Token launches January 1, 2026. Stake $SANG, earn ZABAL multiplier.
**Link:** Latest update → https://paragraph.com/@thezao/zabal-update-3

### Card 4: ZAOOS
**Eyebrow:** The lab
**Body:** The monorepo where new things are prototyped. Forkable for any community: change one config file and you have your own gated music hub.
**Link:** Fork it → https://github.com/bettercallzaal/zaoos

---

## SECTION 3 — Choose your door (3-up grid)

### I'm an artist
- Join the community → /community
- Get into a Fractal → calendar / Discord
- Compete in WaveWarZ → @WaveWarZ on X

### I'm a builder
- Read 200+ research docs → https://github.com/bettercallzaal/zaoos/tree/main/research
- Fork ZAOOS → https://github.com/bettercallzaal/zaoos/blob/main/FORK.md
- Book a call → https://calendly.com/zaalp99/30minmeeting

### I'm a fan
- Watch a battle → https://www.wavewarz.com
- Read the journal → https://bettercallzaal.github.io/bcz-journal/
- Subscribe to the newsletter → https://paragraph.com/@thezao

---

## SECTION 4 — Right now (live stats block)

**WaveWarZ:** 906 battles · 474.87 SOL traded · $706 in artist payouts to date
**The ZAO:** 188+ on-chain Respect holders · daily newsletter unbroken at Day 492+
**ZABAL:** Leaderboard live · token launch Jan 1, 2026
**Built in public:** [github.com/bettercallzaal](https://github.com/bettercallzaal) — 103 repos and counting

_(Numbers update via the live ecosystem map at [bettercallzaal.github.io/bcz-journal/nexus/](https://bettercallzaal.github.io/bcz-journal/nexus/) — keep parity if you can.)_

---

## SECTION 5 — Trusted by / featured artists strip

Same artist showcase block thezao.com already has. No copy change needed — just confirm the lineup matches the homepage:

Jango UU · Songs of Eden · Hurric4n3Ike · AttaBotty · Clejan · NessytheRilla · Jadyn Violet · Maxwell Aden

---

## SECTION 6 — On-chain anchors (collapsible / footer table)

For verifiers. Optional but adds credibility for builders/partners.

| What | Chain | Contract |
|---|---|---|
| OG Respect | Optimism | 0x34cE89baA7E4a4B00E17F7E4C0cb97105C216957 |
| ZOR Respect | Optimism | 0x9885CCeEf7E8371Bf8d6f2413723D25917E7445c |
| ZOUNZ DAO | Base | 0xCB80Ef04DA68667c9a4450013BDD69269842c883 |

Built on [ORDAO](https://github.com/sim31/ordao).

---

## SECTION 7 — Footer (existing) — addition

Add: "Read the public journal → bettercallzaal.github.io/bcz-journal" alongside existing socials.

---

## Notes for the Webflow build

- **The 4-up pillar grid is the most important section.** Get that right and the rest can be progressive.
- **Live stats need to be either (a) hand-updated weekly or (b) wired to wavewarz.info via embed/iframe.** Hand-updated is fine for now; just don't let them go stale past 30 days.
- **The "Choose your door" pattern is the clearest version of what the Nexus should do.** Right now thezao.com/nexus is just nav links. This pattern says "tell me who you are, I'll tell you where to start" which is much higher conversion.
- **Don't promise things that aren't shipping.** "ZAO Stock 2026 in Maine" is announced via the Dec 2 ZABAL update so it's safe to mention. Anything past that should stay off the public Nexus until it's real.
- **Match the rich version.** Live source of truth is `bcz-journal/nexus/README.md` → keep both in sync when the ecosystem changes; the journal version is canonical.

---

## What I'd consider for the redirect path (later, not now)

Once the Webflow nexus is shipping the new copy, set up:
- `nexus.thezao.com` → currently 302 redirects to `thezao.com/nexus`. Keep this.
- `thezao.com/nexus` → make this Webflow page the marketing-facing version.
- `bettercallzaal.github.io/bcz-journal/nexus/` → the live "always current" hacker version. Link to it from the Webflow page footer as "Built in public →".

That gives you the marketing surface (Webflow, ZAO-branded) and the source-of-truth surface (GitHub, builder-credibility) without forcing one to do both.
