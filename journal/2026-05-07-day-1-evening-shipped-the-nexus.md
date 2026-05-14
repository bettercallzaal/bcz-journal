# Day 1, Evening — Shipped the start page

**2026-05-07** _(filename uses the old "Nexus" name — see editor's note below)_

> **Editor's note (later):** The section I called "the Nexus" on this day is now at [`/start/`](../start/) because the canonical **ZAO Nexus** is a separate, much-larger link-hub project that lives in its own repo. I didn't want to step on that name. The post below uses the old name throughout because that's what I called it when I shipped it.

Same day as the journal launch. Pushed two more things: the **Nexus** (now at `/start/`) and the **Projects** index at `/projects/`. They're meant to work as a pair: the start page is the *one-door* version (where do I go?), Projects is the *show me the receipts* version (what have you actually built?).

## Why the Nexus needed to exist

`thezao.com/nexus` right now is six nav links: Home, About, $ZAO, Community, Calendar, Leaderboard. That's not a nexus. That's a header.

If a stranger lands there cold, they don't see WaveWarZ. They don't see ZAOOS. They don't see ZABAL. They don't see what's actually shipping. The whole thing I'm running is invisible from the page that's supposed to be the front door.

So the new Nexus does three things the old one doesn't:

1. **Pillars (4-up):** The ZAO, WaveWarZ, ZABAL, ZAOOS each get a card with a one-sentence "what it is" + a primary action. Now you can see the ecosystem at a glance instead of inferring it from the nav.
2. **Choose-your-door (3-up):** Artist / Builder / Fan. Each gets a tailored next step. This came from realizing the old nexus made every visitor figure out their own path. Most won't.
3. **Live numbers:** 906 battles, $42K traded, $706 paid out, Day ~492 of the newsletter. Numbers are there for credibility — and we already have a public real-time stats site at wavewarz.info, so we should be using it.

The rich version lives on GitHub Pages at [`/start/`](../start/) (was `/nexus/` originally). I also wrote a `webflow-paste.md` next to it with the same content broken into Webflow-shaped section blocks so the marketing-facing nexus on `thezao.com` can be upgraded without rebuilding it.

Two-surface theory: keep `thezao.com/nexus` as the polished marketing entry (and the canonical Nexus repo for builders), and let `journal.bettercallzaal.com/start/` be the live, builder-facing "always current" version. Each does what it's good at.

## Why the Projects index needed to exist

I have 103 repos. The `bettercallzaal-coding-hub` repo I started last March was supposed to be the curated index — but it's stale (it says 71 projects when it should say 103) and it's a Next.js app I'd have to deploy to update. The journal repo can do the same job in pure Markdown for free, with one commit.

So `/projects/` now exists with:

- **A featured 9** — the active stuff someone showing up cold should see first
- **Clusters** — ZAO ecosystem, music, Farcaster Snaps, bots, personal sites, client work, tools, forks
- **The full NEXUS lineage** — V1 → V5.8.5 → ZAONEXUS → ZAOOS, 16 repos in 21 days, the receipts of how I actually arrived at the current architecture

That last one matters more than it looks. Anybody scrolling through 103 repos with cryptic names is going to assume I'm scattered. The lineage shows it's the opposite — it's *one project* that I tried 16 architectural variants of in three weeks and then chose. That's a different story.

## What I noticed today

A few things I don't think I'd have written down without doing this in public:

- **The build is way ahead of the audience.** ZAOOS has the feature inventory of a Series-A startup. WaveWarZ has paid out $706 lifetime. Honest version: the rails are built; we need the trains. That's a more credible story than "we have 1,000+ participants" because it's verifiable, and it explains exactly what the work for the next year is.
- **The version-stack pattern is actually a signature.** I used to think it was sloppy. Looking at it now — 16 NEXUS variants, 8 fractal-bots, 2 ZAIs, 3 newsletter bots — it's a way of *thinking out loud in Git*. Each version is an architectural snapshot. It's not waste; it's how I do search.
- **The daily newsletter is the most underleveraged thing in the portfolio.** 16 months unbroken, never said. Today's the day I start saying it.

## What's queued

- **Custom domain on the journal.** `journal.bettercallzaal.com` → GitHub Pages. Easy DNS change.
- **Wire wavewarz.info live data into the Nexus.** The numbers in the live stats block are hardcoded right now; ideally they refresh from the public dashboard. Not urgent.
- **Inbox enrichment.** I've got a private inbox at `zaal/INBOX.md` from this whole research session — over time, raw notes go there, get sanitized, land here.
- **Nexus on the actual Webflow site.** Section 2 (the 4-up pillar grid) alone is the highest-leverage edit; doing that one section would already be a step-change.

---

*Two posts on Day 1 was not the plan, but the day kept giving things worth writing down.*
