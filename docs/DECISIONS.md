# Decisions

The design lock. Every decision that shapes the platform or a game is recorded here, and the other docs describe the design these decisions produce.

## How to change a decision

1. Don't edit an accepted entry's decision. Add a new entry that supersedes it, and mark the old one `Superseded by D-xxx`.
2. Update every doc the change touches in the same commit.
3. An AI agent never adds or supersedes a decision on its own. It stops, explains the conflict, and asks.

Small clarifications that don't change behavior (typos, a missing route in a table) don't need an entry.

## Product

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-001 | 2026-11-04 | Working title is **Game Night**. | Accepted |
| D-002 | 2026-11-04 | **Same room, always.** Jam Room and Signal Chain depend on it; remote play isn't a goal. | Accepted |
| D-003 | 2026-11-04 | **No accounts, ever.** Sessions are transient. | Accepted |
| D-004 | 2026-11-04 | **Three content modes:** Family, General (the default) and After dark. | Accepted |
| D-005 | 2026-11-04 | Game specs start condensed and are expanded to the full [template](GAME_SPEC_TEMPLATE.md) at the start of that game's phase. | Accepted |
| D-006 | 2026-11-04 | Finish two games end to end before starting a third. Unfinished games show in the gallery behind a status badge. | Accepted |

## Platform

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-010 | 2026-11-04 | The host browser is the single source of game truth; games are pure reducers. No server game loop. | Accepted |
| D-011 | 2026-11-04 | **Supabase only, no Redis.** Supabase Realtime (Broadcast + Presence) is the default transport, behind the `packages/realtime` interface. | Accepted |
| D-012 | 2026-11-04 | **Share results are not MVP.** The `shares` table, `/api/share` and `/s/[id]` are scheduled for Phase 5. The keep-alive table and daily job ship in Phase 1. | Accepted |
| D-013 | 2026-11-04 | Media lives in a new Cloudflare R2 bucket, `game-night`, served from `https://game-night.kylehagerman.dev`. Never the `r2.dev` URL. | Accepted |
| D-014 | 2026-11-04 | Every resume (including crash recovery) goes through the 3-2-1 countdown. | Accepted |
| D-015 | 2026-11-04 | Views are sent in full, never as diffs, and only when they change. | Accepted |
| D-016 | 2026-11-04 | Agent instructions live in a root `CLAUDE.md` only. There is no `AGENTS.md`. | Accepted |

## Versions

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-020 | 2026-11-04 | Use the latest stable release of every tool and library. Versions checked on the npm registry on 2026-11-04 are pinned below; a newer version may replace one without a new entry, as long as the change builds and its tests pass. A new **major** version gets a new entry. | Accepted |

| Tool / library | Version (2026-11-04) | Notes |
| -------------- | -------------------- | ----- |
| Node.js | 26 LTS (26.13.0) | Pinned in `.node-version` and `engines` |
| pnpm | 12.16.1 | Pinned in `packageManager` |
| Turborepo (`turbo`) | 2.11 | |
| Next.js | 16.4 | App Router |
| React / React DOM | 19.3 | |
| TypeScript | 7.0 | See risk note below |
| Tailwind CSS | 4.3 | CSS-first config; no JS preset |
| Zod | 4.6 | |
| Zustand | 5.0 | |
| `@supabase/supabase-js` | 2.119 | Phase 1 |
| Tone.js | 15.1 | Phase 4 |
| Vitest | 5.1 | |
| fast-check | 4.10 | |
| Playwright (`@playwright/test`) | 1.64 | Phase 1 |
| ESLint | 10.14 | Flat config |
| `@aws-sdk/client-s3` | 3.x | For `pnpm upload-media` against R2 |
| `tsx` | 4.23 | Runs the TypeScript scripts |

**TypeScript 7 risk.** TypeScript 7 is the native (Go) compiler. If Next.js 16.4, ESLint's TypeScript parser or Vitest can't use it yet, step 0.1 falls back to the latest 6.x and records that here as a new entry.

## Games

### Slide Roulette

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-100 | 2026-11-04 | The presenter gets one Skip per talk: the current slide turns blank for its remaining time, and the Picker carries on. | Accepted |
| D-101 | 2026-11-04 | The presenter sees "up next" on their own phone. | Accepted |
| D-102 | 2026-11-04 | The Picker sees each option's raw content, then a slide preview after choosing. | Accepted |
| D-103 | 2026-11-04 | The MVP rotates photo and text slides; Chart slides come later. | Accepted |

### Signal Chain

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-110 | 2026-11-04 | Roles use neutral names (Seer, Relay, Operator). | Accepted |
| D-111 | 2026-11-04 | The Seer sees the Operator's live progress by default (`seerSeesProgress`). | Accepted |

### Jam Room

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-120 | 2026-11-04 | A pure toy first, with no goals or scoring. Light group challenges can come later. | Accepted |
| D-121 | 2026-11-04 | MVP sounds come from Tone.js synths only; sampled instruments need a checked license first. | Accepted |

### Gap Years

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-130 | 2026-11-04 | The guess timer defaults to 30 s, configurable from 15 to 60 s (`guessMs`). | Accepted |
| D-131 | 2026-11-04 | Every event carries a `source` URL; the validator rejects events without one. | Accepted |

### Story Relay

| ID | Date | Decision | Status |
| -- | ---- | -------- | ------ |
| D-140 | 2026-11-04 | Non-competitive, with no storyteller vote. | Accepted |
| D-141 | 2026-11-04 | The whole game works with zero transcripts; the mic is optional and off by default. | Accepted |
