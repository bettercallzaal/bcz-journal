# Day 1 — Building the Journal

**2026-05-07**

I asked Claude to crawl all 103 of my GitHub repos, do a deep online research pass on The ZAO, WaveWarZ, and the ZABAL stack, and then we kept going. By the end of the session it felt like the right moment to actually point a public repo at this — somewhere I can keep dumping raw notes and have it accumulate into something useful for anyone trying to figure out what I'm building. So this is that.

## What we found

### My GitHub footprint, by the numbers

- **103 repos.** 91 public, 12 private, 6 forks.
- **27 pushed in the last 30 days.** Another 27 in the 30–90 day window. About 36 are dormant past 6 months.
- Default stack across 50+ repos: **Next.js + Tailwind + Vercel + Supabase + Farcaster.** Bots in Python.
- One signature workflow pattern: I version-stack instead of branching. `NEXUSV1` → `NEXUSV5.8.5` was 16 separate repos created in 21 days last July. The descendant of that whole arc is **ZAOOS**, the current monorepo where everything new gets prototyped.

### The lab → graduate model is working

ZAOOS is what I call "the lab" — a monorepo with 100 branches and ~4200+ files. Its README documents an explicit graduation criterion: a thing leaves ZAOOS and gets its own repo + DB + domain when it's production-ready, ready to share publicly, and ready to attract new users. **CoC Concertz** has graduated. **FISHBOWLZ** was its own thing then paused. **ZAOstock** is mid-graduation.

The pattern works because the cost of starting a new ZAO experiment in ZAOOS is near zero, and the cost of graduating it (own repo, own DB, own domain) is high enough that it forces a real decision.

### The research library is bigger than I remembered

ZAOOS has **202 active research docs + 79 archived**, organized across 13 topic folders: agents, music, dev-workflows, infrastructure, governance, community, cross-platform, farcaster, identity, business, wavewarz, security, events, plus inspiration. ~500K+ words. It already has a good index at `research/README.md` — feature scoreboards, "built vs to-do" tables, key starting points by use case. It's been mostly invisible because it's buried inside ZAOOS. I want to surface it from here.

→ See [research/](../research/) for the public index.

### The ZAO has the rails; now we need the trains

WaveWarZ live stats today (from [wavewarz.info](https://wavewarz.info)):
- 474.87 SOL traded ($42,107) across 906 battles
- Artist payouts to date: 7.96 SOL ($706)
- Platform revenue: 3.75 SOL ($333)

The infrastructure works. The volume hasn't hit the level where artist economics get materially changed by being on it. The honest framing of the project right now is: rails built, trains needed.

### "Year of the ZAO" is at Day 492 today

I've published every single day on Paragraph since January 1, 2025. That's the most underrated thing in my portfolio and I've never said it that way out loud. Going to start.

## What I'm setting up next

- **`research/`** in this repo will mirror the ZAOOS research index so people can browse it without spelunking through a 4000-file monorepo.
- **`ecosystem/`** will hold a current public map of The ZAO + WaveWarZ + the supporting projects so I have one place to point people instead of seven.
- **`journal/`** is here — daily-ish entries when there's something worth writing down. Won't be every day. Will be when something matters.

## What I'm leaving out

This repo is public-by-default, so things that aren't ready to be public stay in private working files. Things that aren't here intentionally:
- Internal strategy framing
- Anything still in legal evaluation
- Specific people's names in contexts they haven't consented to
- Anything that could let someone front-run a launch

If something looks like it's missing from a story, that's probably why.

---

*Built today with the help of [Claude Code](https://claude.com/claude-code) — this whole session was the kind of "Claude crawls everything, I read what's interesting, we decide what to do next" workflow that's quietly become how I operate.*
