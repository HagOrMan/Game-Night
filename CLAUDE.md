# Game Night

A Next.js site hosting small party games for a group of friends in one room: a TV runs the game, and phones join as controllers. The project is a pnpm + Turborepo monorepo: `apps/web`, shared `packages/*`, and one package per game under `games/*`.

## Read before working

1. `docs/IMPLEMENTATION_PLAN.md`: find the current phase and step, and work only on that step.
2. `docs/DECISIONS.md`: the locked decisions and version pins.
3. `docs/ARCHITECTURE.md`: the engine, the monorepo layout and the stack.
4. The doc for the area you're touching: `docs/PROTOCOL.md` (rooms, messages), `docs/SITE_AND_UI.md` (routes, UI kit, theming), `docs/CONTENT_GUIDE.md` (content modes, ratings).
5. For game work: `games/<slug>/SPEC.md`. It is that game's source of truth.

## The design is locked

- Build what the docs say. If the docs are silent, ambiguous or wrong for what you're building, stop and ask. Don't pick an answer and carry on.
- Never add, change or supersede an entry in `docs/DECISIONS.md` without the user's go-ahead. When one changes, update every doc it touches in the same change.
- A game spec in condensed form is expanded to the full `docs/GAME_SPEC_TEMPLATE.md` (and confirmed with the user) before that game's code is written.
- Tick the "Done when" checkboxes in `docs/IMPLEMENTATION_PLAN.md` as items are verified, not when code is merely written.

## Hard rules in code

- A game's `logic.ts` is a pure reducer: no `Date.now()`, no `Math.random()`, no fetches, no React, no DOM or browser APIs. Time comes from `ctx.now`, randomness from `ctx.rng`.
- `packages/engine` has no React or DOM imports.
- A game imports only shared packages, never another game and never `apps/web`.
- Only `packages/realtime` imports Supabase or touches transports. Games and the app use the `Transport` interface.
- Every message crossing the network is parsed with its Zod schema before use. Changing a message shape bumps the protocol version.
- Views are full, never diffs, and are sent only when they change. Nothing is sent on a tick.
- Content is filtered only through `pickContent`. Every content entry has a `rating`.
- Media URLs go through `packages/media`. Never hard-code the R2 domain or use `r2.dev`.
- Secrets never go in `NEXT_PUBLIC_*` variables. R2 credentials never go to Vercel.

## Running commands

This overrides the user's global "never run builds or tests" rule for this repo:

- You may run installs, builds, linters, type checks and tests, but **confirm with the user first**: name the suites or commands you'll run and what they need installed.
- Adding packages as repo dependencies is normally fine. Anything installed globally or outside the repo (Node versions, browser binaries, CLIs) needs explicit approval first.
- Use the latest stable version of a library when adding it (check the npm registry), within the policy in `docs/DECISIONS.md` (D-020).
- Use pnpm, never npm or yarn, for anything in the repo.
