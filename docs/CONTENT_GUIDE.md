# Game Night — Content Guide

Rules for every prompt, line, word, event and photo a game deals. Locked: changes go through [DECISIONS.md](DECISIONS.md).

## Content modes

Every room plays in one of three content modes, chosen by the host in the lobby: Family, General (the default) and After dark. The mode filters everything a game deals.

| Mode              | What's in it                            | Draws entries rated                                              |
| ----------------- | --------------------------------------- | ---------------------------------------------------------------- |
| Family            | Safe for kids and grandparents          | family                                                           |
| General (default) | What any adult would say at a party     | family, general                                                  |
| After dark        | Explicit, dark, suggestive; adults only | family, general, after-dark (about 60% of draws from after-dark) |

- `SetupCtx` carries `contentMode`, and `pickContent(pool, mode, rng)` in `packages/content` does the filtering, so games never filter by hand.
- Each game lists the modes it supports in `game.json` (`contentModes`). A game with a single mode hides the picker. A game that deals no content (Signal Chain, Jam Room) lists none and hides the picker.
- Choosing After dark asks the host to confirm that everyone in the room is an adult. The mode shows as a badge on the TV and on every phone.
- The mode changes only between rounds, from the lobby or the pause menu.

## Ratings and files

- Every content entry carries `rating: 'family' | 'general' | 'after-dark'`.
- After-dark entries live in separate `*.after-dark.json` files so they're easy to review or leave out.
- Each content file has a Zod schema next to it in the game's `src/content/`.

## Validation (`pnpm validate-content`)

Runs in CI and before every commit that touches content. It rejects:

- entries without a `rating`, or with a rating that doesn't match the file (an after-dark entry outside a `*.after-dark.json` file, or the reverse);
- entries that fail their file's schema;
- duplicate ids within a game;
- game-specific rules a game registers, such as Gap Years' `source` URL and year range.

## Photos

- In After dark, games use only photos tagged `people: false` (animals, objects, scenes) or AI-generated people from the R2 bucket. Pexels' guidelines don't allow identifiable people to appear in a bad light or an offensive way, so real people never sit next to after-dark lines.
- Every Pexels search term is tagged `people: true | false`, and each pooled photo inherits it.
- Pexels attribution: a prominent "Photos provided by Pexels" link on the game page and end screen, and a small "Photo: Name on Pexels" credit on each slide that shows one. AI images show "AI-generated". See the [Pexels API docs](https://www.pexels.com/api/documentation/) and [request limits](https://help.pexels.com/hc/en-us/articles/900005852323).
- Check an image generator's terms before using its output.

## Free text

Free-text games (Fit Check's Name the fit, Bluff Fill, Majority Rules, Who Said It) mask profanity in Family mode only.

## Writing content

- Drafting packs with AI is fine; every pack gets a human read-through before it's committed, after-dark packs especially.
- Facts (dates, numbers) are verified against a cited source, never taken from an AI draft as-is.
