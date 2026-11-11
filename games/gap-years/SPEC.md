# Gap Years — SPEC

Status: idea · Slug: `gap-years` · Last updated: 2026-11-04 · Condensed; expand to the full [template](../../docs/GAME_SPEC_TEMPLATE.md) before Phase 1.

## Pitch

Two moments from history appear on screen and everyone guesses, on their phone, how many years apart they were. Scoring is relative, so being 20 years off on a 2,000-year gap is great and 20 years off on a 30-year gap is not. Five rounds, then a winner is crowned. This is the simplest networked game and the best one to build first.

## At a glance

| Field          | Value                                          |
| -------------- | ---------------------------------------------- |
| Modes          | Party                                          |
| Players        | 2–12                                           |
| Session length | About 1.5 min per round; 8–12 min for 5 rounds |
| Devices        | Host screen; phones                            |
| Late join      | Allow; starts with 0 points                    |
| Key roles      | None                                           |
| Content modes  | Single mode; picker hidden (see open questions) |

## How to play

<!-- howto:start -->

Two events appear, for example "The Ming dynasty is founded" and "India's first feature film premieres". How many years apart were they? Type your guess before the timer ends. The closer you are relative to the real gap, the more points you get, and nailing it earns a bullseye bonus. Bonus: say which happened first. After 5 rounds the top score wins.

<!-- howto:end -->

That example is 1368 to 1913, a 545-year gap.

## Flow

| Phase    | Host screen                                                                                             | Phones                                                                | Ends when                  | Next               |
| -------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------- | ------------------ |
| `intro`  | Round N of 5; the two events appear one at a time, in random order                                      | "Get ready"                                                           | 4 s                        | `guess`            |
| `guess`  | Both events, timer, who has locked in                                                                   | Number pad, nudge buttons (±1, ±10, ±100), "which came first?" toggle | All locked in or `guessMs` | `reveal`           |
| `reveal` | Events slide onto a timeline; the gap counts up; everyone's guesses drop as pins on a ruler; a fun fact | Your guess, the answer, your points                                   | 10 s or host taps          | `scores`           |
| `scores` | Animated leaderboard                                                                                    | Your rank                                                             | 5 s                        | `intro` or `final` |
| `final`  | Podium and crowned winner; best single guess                                                            | Your final place                                                      | Host                       | `lobby`            |

## Settings

| Key | Default | Range |
| --- | ------- | ----- |
| `rounds` | 5 | 3–15 |
| `guessMs` | 30000 | 15000–60000 |
| `difficulty` | mixed | mixed, easy, hard |
| `regions` | all | multi-select |
| `categories` | all | multi-select |
| `scoring` | relative | relative, rank |
| `orderBonus` | — | on, off |

## Scoring (relative mode)

```latex
\text{relErr} = \frac{|g - a|}{\max(a,\ 20)} \qquad \text{points} = \operatorname{round}\big(1000 \cdot \max(0,\ 1 - \text{relErr})^2\big)
```

Here g is the guess and a the actual gap. The floor of 20 keeps tiny gaps from being brutal; squaring rewards being close more than being roughly right. Extras: +300 bullseye when within 1% (at least 1 year), +150 for the closest guess in the round (shared on ties), +100 for the correct order when `orderBonus` is on. No guess scores 0.

| Guess for a 545-year gap | Off by | Base points         |
| ------------------------ | ------ | ------------------- |
| 545                      | 0      | 1000 + 300 bullseye |
| 500                      | 45     | 842                 |
| 300                      | 245    | 303                 |
| 1000                     | 455    | 27                  |

**Rank mode** (setting): each round awards N, N−1, … points by closeness rank. Final ties break on the lower total of relative errors.

## Content: `events.json`

```ts
interface HistEvent {
  id: string; // 'ming-founded'
  title: string; // 'The Ming dynasty is founded in China'
  year: number; // astronomical numbering: 1 BCE = 0, 44 BCE = -43
  display: string; // '1368 CE'
  precision: "exact" | "circa";
  region:
    | "east-asia"
    | "south-asia"
    | "middle-east"
    | "africa"
    | "europe"
    | "americas"
    | "oceania"
    | "global";
  category:
    | "politics"
    | "science"
    | "tech"
    | "culture"
    | "sports"
    | "exploration"
    | "pop";
  difficulty: 1 | 2 | 3;
  fact?: string; // one line shown at reveal
  source: string; // URL used to verify the year
}
```

- **Point events, not eras.** "The Industrial Revolution" spans decades; use a pinned moment such as "James Watt patents his improved steam engine (1769)".
- **No year zero.** Storing astronomical years makes gaps a plain subtraction across BCE/CE: 44 BCE to 2000 CE is 2,043 years.
- **Verification is mandatory.** Draft events with AI if you like, but AI-drafted dates are often a few years off. Each entry needs a `source` URL, and `pnpm validate-content` rejects entries without one, duplicate ids, or years outside −3000 to the current year.
- MVP target: 150 verified events. Launch target: 300 events across all regions, which supports thousands of distinct pairs.

## Pair picker (seeded)

Gaps of at least 5 years; spread across buckets (under 50, 50–200, 200–800, over 800 years) so a game doesn't get five similar gaps; `circa` events only in pairs with gaps over 100 years; no event repeats within a game; difficulty is the max of the pair.

## Timers and pause

`guessMs` freezes; locked-in guesses stay locked; the nudge buttons are disabled while paused. Reveal animations pause mid-motion via `data-paused`.

## Edge cases

- A phone reconnects mid-guess and gets its unsent draft back from localStorage.
- A guess over 10,000 is clamped with a gentle "that's before writing" note.
- One player: allowed as a solo practice run (overrides the 2-player minimum).

## Scope

**MVP:** 5 rounds, relative scoring, 150 verified events, timeline reveal, podium.

**Later:** order bonus, themed packs ("music history", "space race"), a Screen-mode pass-the-phone variant, and a sister game "Map Drop" that reuses the scoring for distances.

## Decisions

D-130 and D-131 in [DECISIONS.md](../../docs/DECISIONS.md#gap-years).

## Open questions

Settle these when the spec is expanded at the start of Phase 1:

- **Content mode and ratings.** The content guide says every entry carries a `rating`, but `HistEvent` has no `rating` field. Either add `rating` (probably all `family`) and list `["family"]` in `game.json`, or exempt this game from the rating rule.
- **Players.** At a glance says 2–12, but edge cases allow a 1-player practice run. Decide whether `players.min` is 1.
- **`orderBonus` default.** The settings list it as on/off, the scoring rules use it, and the MVP lists the order bonus as Later. Pick a default (probably off for the MVP).
