# Inbox

Raw drops. Anything goes — tweets, transcripts, screenshots-of-text, half-thoughts, podcast quotes, decisions made over coffee. Dump fast; process later.

## How this works

```
You drop content here
        │
   Claude processes
        │
   ┌────┼────────────────┐
   ▼    ▼                ▼
journal/  zaoos/research/  nexus + ecosystem
(dated   (topic-based,    (latest-update
 entry)  if substantive)   sections refresh)
```

The fan-out pattern:

1. **Drop** — paste raw content into a new file under `inbox/` (use any filename, dated optional).
2. **Process** — when Claude runs through the inbox, each piece gets routed:
   - **Voice-y, dated, observational** → becomes a [journal](../journal/) entry
   - **Topic-deep, substantive research** → also lands in the [ZAOOS research library](https://github.com/bettercallzaal/zaoos/tree/main/research) under the right topic folder
   - **Major announcement / new project / new collaborator** → triggers an update to the [Nexus](../nexus/) and [Ecosystem](../ecosystem/) pages
3. **Archive** — the raw drop moves to `inbox/_archive/YYYY-MM-DD-<slug>.md` so the trail is preserved
4. **Publish** — the journal entry / research doc / nexus update is committed

## What goes where

| Type of content | Goes to | Notes |
|---|---|---|
| Day's thoughts | `journal/` | Dated, voice-y |
| Reflection on shipping something | `journal/` | Why it mattered |
| Deep dive on a tool / API / pattern | `zaoos/research/` | Becomes part of the canonical library |
| New project announcement | `nexus/` updates + `journal/` post | Both surfaces refresh |
| New collaborator | `ecosystem/` update | Add to the people section |
| Live stat changes | `nexus/` "Right now" block | Hand-update for now |
| Sensitive / not-yet-public | NOT here | Use `~/coding/bots1/zaal/INBOX.md` (private) instead |

## What this folder is NOT for

- Code. Code goes in the actual repos.
- Anything sensitive. This folder is public via GitHub Pages.
- Anything you want hidden from search engines. It's all crawled.

If in doubt: dump it in private inbox first (`zaal/INBOX.md` outside this repo), let Claude do a sanitize pass, then it lands here.

## Drop pattern

A new drop is just a markdown file. Optional first line for context:

```
> source: <where this came from — twitter, transcript, my own notes>
> date: <when, if known>

[content]
```

If you don't add a header, Claude will guess from content.
