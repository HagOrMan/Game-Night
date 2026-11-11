# Game Night — Site and Shared UI

The website flow, gallery metadata, and the shared UI kit. Locked: changes go through [DECISIONS.md](DECISIONS.md).

Three screens matter: the gallery, a game page with its how-to-play, and the play surface. Phones skip all of it and land straight on the controller from a QR code.

## Routes

| Route                         | Shows                                                                                                                                                 |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/`                           | Gallery: a card per game with a looping preview, tagline, mode badges, player count, minutes; filter by mode and player count                         |
| `/games/[slug]`               | Preview screenshots, How to play, what you need (TV + phones, one screen, mic), and start buttons: "Host on this screen" and/or "Play on this screen" |
| `/games/[slug]/play`          | Full-screen play surface: lobby (with the content mode picker), then the game                                                                         |
| `/games/[slug]/showcase?shot=` | Renders the host screen frozen in one of the game's showcase states; used by `pnpm screenshots`                                                      |
| `/join`                       | Type any room code                                                                                                                                    |
| `/j/[code]`                   | Phone controller                                                                                                                                      |
| `/s/[id]`                     | Shared results, read-only; expires after 30 days (later)                                                                                              |

## Gallery metadata (`game.json`)

```json
{
  "slug": "slide-roulette",
  "name": "Slide Roulette",
  "tagline": "Present a deck you've never seen. Your friend picks the slides.",
  "modes": ["party", "hybrid"],
  "players": [2, 12],
  "minutes": [15, 25],
  "needs": ["tv", "phones"],
  "tags": ["improv", "performance"],
  "contentModes": ["family", "general", "after-dark"],
  "status": "playable",
  "accent": "#2F6FEB"
}
```

`status` is `idea | prototype | playable` so unfinished games can sit in the gallery behind a "Coming soon" badge. `game.json` has a Zod schema in `packages/engine`; the build fails on an invalid file.

## How-to-play: one source

The player-facing rules live once, inside each game's SPEC.md between `<!-- howto:start -->` and `<!-- howto:end -->`. A build step extracts that block for the game page, the pause menu and an optional lobby tutorial, so docs and game can't drift apart.

## Previews without manual screenshots

Each game exports a few `showcase` states: seeded, frozen moments such as the cover shot or a reveal. `/games/[slug]/showcase?shot=cover` renders the host screen with that state, and `pnpm screenshots` uses Playwright to capture a 1600×900 WebP per shot plus an optional 6-second WebM loop for hover previews. Re-run it whenever a game's look changes. Hand-taken screenshots dropped into the game's `media/` folder also work.

Screenshots and preview loops land in the game's `media/` folder, and `pnpm upload-media` publishes them to R2. The gallery loads them from `game-night.kylehagerman.dev`, so they never bloat the repo or the Vercel deploy. Until a game has media, its card shows a generated placeholder in its accent color.

## Shared UI kit (`packages/ui`)

| Component                  | Notes                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------ |
| Lobby                      | Code + QR, player list, settings form generated from the config schema, content mode picker, start button |
| AvatarPicker, PlayerChip   | Emoji or generated avatars, player color                                             |
| Countdown                  | 3-2-1 used for game start and every resume                                           |
| PauseOverlay, PauseMenu    | Same on host and phones                                                              |
| TimerRing, TimerBar        | Driven by `useTimer`, pause-aware                                                    |
| Scoreboard, Podium         | Animated rank changes, final winner reveal                                           |
| InfoSheet                  | Renders the extracted how-to-play                                                    |
| BigButton, SubmitLock      | Phone controls; "Locked in, waiting for 3 others"                                    |
| AudioUnlock                | Tap-to-enable prompt; browsers block audio until a tap                               |
| WakeLock, FullscreenToggle | Keeps host and phones from sleeping                                                  |

## Theming

Base tokens are CSS variables (`--gn-bg`, `--gn-fg`, `--gn-accent`, `--gn-font-display`, `--gn-radius`), defined once in the Tailwind v4 theme in `packages/config`. Each game's `theme.css` overrides them under `[data-game="<slug>"]` and brings its own fonts, backgrounds and sound pack, so shared components automatically pick up each game's look. A game can also replace any shared component outright.

## Device ergonomics

- **Host screen:** design for 1920×1080 viewed from about 3 m; body text at least 32 px; nothing important in the outer 5% (TV overscan).
- **Phones:** portrait, one hand, touch targets at least 48 px, no scrolling during play, haptic tap where supported.
- **Accidents:** the host asks before leaving the page; phones disable pull-to-refresh on the controller.
