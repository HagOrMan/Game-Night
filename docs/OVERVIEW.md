# Game Night — Overview

One Next.js site hosts a growing shelf of small party games for a group of friends. Every game plugs into one shared engine for rooms, timers, pause/resume and scoring, and keeps its own look.

This is the product-level design. It is locked: changes go through [DECISIONS.md](DECISIONS.md).

## Goals

- A new game is a folder: a SPEC.md, a pure state reducer, and a few screens.
- One link to play: open the site, pick a game, playing in under 30 seconds. No accounts.
- Pause anything at any time without breaking timed games; resume with a 3-2-1 countdown.
- Free to run at friend-group scale on hobby tiers.
- Each game's SPEC.md is the source of truth that a person or an AI coding agent can build the game from on its own.

## Non-goals (for now)

Public matchmaking, accounts, moderating strangers, monetization, anti-cheat beyond "friends don't peek", native apps, remote play.

## Assumptions

- Everyone plays in the same room, around one TV.
- Sessions are transient: no accounts and no history.
- The only thing that will outlive a game is an optional share-results link that expires after 30 days. This is not part of the MVP (see [D-012](DECISIONS.md)).

## Play modes

Every game declares one or more of these:

| Mode   | Devices                                               | Where game state lives           | Network          | Fits                                                                 |
| ------ | ----------------------------------------------------- | -------------------------------- | ---------------- | -------------------------------------------------------------------- |
| Screen | One shared screen (TV or laptop), everyone around it  | That browser only                | None             | Story Relay, Forehead                                                |
| Party  | One host screen + each player's phone as a controller | Host browser, mirrored to phones | Realtime channel | Gap Years, Slide Roulette, Signal Chain                              |
| Hybrid | Plays as Screen; phones can join for extra roles      | Host browser                     | Optional         | Jam Room (solo keyboard or phones), Slide Roulette audience reactions |

## Glossary

| Term        | Meaning                                                                           |
| ----------- | --------------------------------------------------------------------------------- |
| Host screen | The big shared display; runs the game and owns the truth                          |
| Controller  | A player's phone joined to a room                                                 |
| Room        | One live session of one game, identified by a 4-letter code                       |
| Phase       | A named step of a game (lobby, guessing, reveal...)                               |
| Intent      | Something a player asks to do ("submit guess 340"); the host decides if it counts |
| View        | The slice of state a device is allowed to see and render                          |
| Content mode | Family, General or After dark; filters everything a game deals (see [CONTENT_GUIDE.md](CONTENT_GUIDE.md)) |

## The games

| Game | Mode | Spec | Phase |
| ---- | ---- | ---- | ----- |
| Story Relay | Screen | [games/story-relay/SPEC.md](../games/story-relay/SPEC.md) | 1 |
| Gap Years | Party | [games/gap-years/SPEC.md](../games/gap-years/SPEC.md) | 1 |
| Slide Roulette | Party, Hybrid | [games/slide-roulette/SPEC.md](../games/slide-roulette/SPEC.md) | 2 |
| Signal Chain | Party | [games/signal-chain/SPEC.md](../games/signal-chain/SPEC.md) | 3 |
| Jam Room | Party, Screen | [games/jam-room/SPEC.md](../games/jam-room/SPEC.md) | 4 |

More candidates are in [IDEAS.md](IDEAS.md).

## Where everything else lives

| Doc | Covers |
| --- | ------ |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Stack, hosting, media, monorepo layout, the game engine, data storage |
| [PROTOCOL.md](PROTOCOL.md) | Room codes, channels, join/reconnect, message envelope, message budget |
| [SITE_AND_UI.md](SITE_AND_UI.md) | Routes, gallery metadata, how-to-play, previews, shared UI kit, theming, ergonomics |
| [CONTENT_GUIDE.md](CONTENT_GUIDE.md) | Content modes, ratings, validation, photo rules |
| [GAME_SPEC_TEMPLATE.md](GAME_SPEC_TEMPLATE.md) | The shape every game SPEC.md follows |
| [DECISIONS.md](DECISIONS.md) | Every locked decision, and how to change one |
| [IMPLEMENTATION_PLAN.md](IMPLEMENTATION_PLAN.md) | Phases, steps, playtest gates, testing, risks |
| [IDEAS.md](IDEAS.md) | Future games and shared systems |
