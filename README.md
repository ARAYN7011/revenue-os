# Revenue OS

**A daily work list for sales reps: every morning, each account executive opens one screen that says what to do today, in order, and why.**

> Built at [Fello](https://hifello.com) as the lead engineer. This repository is a write-up: the code is private, the thinking is here.

▶ **[Watch the 3-minute walkthrough](https://drive.google.com/file/d/1mnus7z_RkMBJGuaosIH7K0V1gGZqxAvF/view)**

---

## The problem

A sales rep's CRM holds thousands of accounts, call recordings, emails and meetings. Reps don't need more data. They need to know the next best action by 9 a.m. Most "AI for sales" tools answer with a chat box. Reps don't want to ask; they want to be told.

## What it does

Each night (and through the day) a pipeline reads the warehouse, builds a dossier per account, and asks a model for two things:

1. **Pass 1, a brief per account:** what's happening, what was promised, what changed.
2. **Pass 2, the rep's ranked day:** three kinds of work, deliberately *not* peers:
   - **The calendar:** meetings the rep is actually taking. Unranked; it *is* the day.
   - **Obligations:** a promise made on a call, an email awaiting reply, a quote about to expire.
   - **Best accounts to work:** the residue of the day, chosen for a stated, measured reason.

Managers get team views. Admins can "view as" any rep. An MCP server exposes the same capabilities to agents.

```
HubSpot mirror ─┐
Call transcripts ─┤→ dossier per account → Pass 1 brief → Pass 2 ranked day → Postgres → Next.js app (/today)
Calendar ───────┤                                                     ↘ MCP server (agents)
Web signals ────┘
```

## Design decisions worth reading

- **Measure first, design second.** The product's ordering rule comes from one measured asymmetry: stale *meetings* are dead pipeline, stale *quotes* are alive. Every claim in the design doc carries the query that produced it and the date it was measured.
- **The data lies in specific ways, and the notes say how.** Example: an association column held only the *primary* link, so the biggest umbrella company in a book looked like a 3-contact account while driving ~94% of one query's cost. Writing down how each fact was measured is why the notes were still true a month later.
- **Match people by email, never domain.** A third of external meetings were on free-mail addresses, because real-estate agents run on them. Domain matching would have lost a third of the day.
- **Prompts live in a file, not in code.** The ranking rules are in a reviewable `PROMPTS.md`; code is generated from it.
- **Architecture enforced by a ratchet.** The app is split by what code *is* (routes, views, capabilities, agents, platform). A boundary checker runs in tests, and its violation count can go down but never up.
- **One run per rep.** One cron fire fans out to every rep with open deals, under shared cost bounds, with every read and write scoped to that rep.
- **A rebuild started from a data dictionary, not the old design.** v2 began with only what was *measured* about the data. Responses to that data were dropped and redesigned.

## Stack

TypeScript · Next.js 15 · pnpm monorepo (`web`, `pipeline`, `shared`, `db`) · Postgres/Supabase · OpenRouter models · MCP · HubSpot app · Railway · Vitest + structural tests · ~1,700 commits

## What I'd highlight

| Skill | Evidence |
|---|---|
| Turning messy CRM data into a decision | Dossier → brief → ranked day pipeline |
| Making LLM output trustworthy | Prompts as reviewable artifacts; ranking rules outside code; evidence on every claim |
| Keeping a fast-moving codebase navigable | Layer boundaries with an enforcing ratchet; a context file per layer for coding agents |
| Cost control | Per-run bounds; excluding one umbrella account cut a query from ~30 s to ~130 ms |

---

*Author: [Aryan Pasreja](https://www.linkedin.com/in/aryan-pasreja/) · AI engineer*
