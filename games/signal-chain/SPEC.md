# Signal Chain — SPEC

Status: idea · Slug: `signal-chain` · Last updated: 2026-11-04 · Condensed; expand to the full [template](../../docs/GAME_SPEC_TEMPLATE.md) before Phase 3.

## Pitch

A team of three passes instructions down a broken chain: one sees the target but can't speak, one can speak but can't hear, and one does the task on a blacked-out phone. The app itself enforces the senses: it hides the target, blanks the Operator's screen and plays noise into the Relay's headphones.

Roles get neutral names (Seer, Relay, Operator) so the game reads as a communication challenge rather than jokes about disabilities.

## At a glance

| Field          | Value                                                          |
| -------------- | -------------------------------------------------------------- |
| Modes          | Party                                                          |
| Players        | 3–9; best at 3 or 6 (two teams)                                |
| Session length | About 2 min per task; 15–20 min for a full rotation            |
| Devices        | Host screen; a phone each; headphones for the Relay (optional) |
| Late join      | Spectate, then rotate in                                       |
| Key roles      | Seer, Relay, Operator (any disconnect auto-pauses)             |
| Content modes  | None (deals no content; picker hidden)                         |

## How to play

<!-- howto:start -->

Three of you form a chain. The Seer sees the target on their phone but can't talk; gesture only. The Relay watches the Seer and talks to the Operator, but wears headphones full of noise, so they can't hear questions back. The Operator follows the Relay's words on a phone whose screen goes completely black. Finish before the timer runs out. Then swap roles until everyone has done each one.

<!-- howto:end -->

## Roles

| Role       | Can                                                   | Can't                              | Phone shows                                                        |
| ---------- | ----------------------------------------------------- | ---------------------------------- | ------------------------------------------------------------------ |
| Seer       | See the target and the Operator's live progress; hear | Speak                              | Target + live overlay of the Operator's input; "Lock it in" button |
| Relay      | See the Seer; speak                                   | Hear (app plays noise)             | "Put headphones on", noise volume, timer                           |
| Operator   | Hear; touch                                           | See (screen is black, eyes closed) | Black screen that still takes input; click sound per touch         |
| Spectators | Watch                                                 | —                                  | Target + progress, so they can enjoy the chaos                     |

**Seating:** the Seer faces the Relay; the Operator sits with eyes closed, ideally facing away from the TV. The host screen shows the timer, scores and the Operator's touches drawn live, but never the target, so the Relay can glance at it without cheating.

## Tasks

Each is a small minigame module on the Operator's phone with a target generator and a scorer:

| Task         | Operator does                               | Target                      | Accuracy measure                         |
| ------------ | ------------------------------------------- | --------------------------- | ---------------------------------------- |
| Grid Tap     | Tap cells of an invisible 3×3 grid in order | 3–6 cells                   | Correct cells in correct order           |
| Swipe Combo  | Swipe up/down/left/right                    | 4–7 swipes                  | Longest matching run                     |
| Dial         | Turn a knob with one finger                 | An angle, like 4 o'clock    | Angle error                              |
| Sliders      | Set 3 vertical sliders                      | Heights in thirds           | Mean height error                        |
| Pattern Lock | Join dots on a 3×3 grid                     | A 4–6 dot path              | Matching edges                           |
| Trace        | Draw a shape                                | Star, house, spiral, letter | Shape match score (unistroke recognizer) |
| Rhythm       | Hold and tap in time                        | "Hold 3 beats, tap twice"   | Timing error                             |

Difficulty sets the target length and the task pool. A three-finger tap clears the Operator's input. The Seer ends the attempt with "Lock it in", or the timer does.

## Flow

| Phase     | Host screen                                            | Phones                                                | Ends when           | Next                          |
| --------- | ------------------------------------------------------ | ----------------------------------------------------- | ------------------- | ----------------------------- |
| `lobby`   | Players, team split                                    | Join                                                  | Host starts         | `roles`                       |
| `roles`   | Who is Seer, Relay, Operator; seating tips             | Role card with instructions; Relay starts noise check | All tap "Ready"     | `task`                        |
| `task`    | Timer + live Operator touches                          | Per role, as above                                    | Lock-in or `taskMs` | `reveal`                      |
| `reveal`  | Target overlaid on the attempt, accuracy %, points     | Same reveal                                           | 8 s or host taps    | `roles` (rotate) or `results` |
| `results` | Team and individual scores; best Seer, Relay, Operator | —                                                     | Host                | `lobby`                       |

## Settings

| Key | Default | Range |
| --- | ------- | ----- |
| `taskMs` | 90000 | 45000–180000 |
| `difficulty` | — | easy, medium, hard |
| `tasks` | all MVP tasks | multi-select from the task list |
| `teams` | auto | auto, 1, 2 |
| `relayNoise` | on | on, off |
| `noiseVolume` | — | 0–1 |
| `rotationsPerPlayer` | 1 | 1–3 |
| `seerSeesProgress` | on | on, off |

## Scoring

```latex
\text{points} = \operatorname{round}\Big(1000 \cdot \text{accuracy} \cdot \big(0.5 + 0.5 \cdot \tfrac{\text{timeLeft}}{\text{taskMs}}\big)\Big)
```

Worked example: 80% accuracy with 30 s left of 90 s gives 1000 × 0.8 × (0.5 + 0.167) = 533 points. The team scores the points; each player also banks them under the role they played, which drives the "best Seer" awards. With two teams, teams alternate turns and compete on total.

## Network

The Operator's phone streams input to the host in batches (at most 5 messages per second) for the live drawing. The target lives only in the Seer's and spectators' private views. Scoring runs on the host from the final input.

## Timers and pause

Pausing freezes `taskMs`, stops the Relay's noise (so they can hear the room), and keeps the Operator's partial input. After the countdown the noise fades back in during the "3", so it's at full volume by "1".

## Tech

Pointer Events with `touch-action: none` on a full-screen black canvas; Fullscreen and Screen Wake Lock APIs; a brown-noise generator in Web Audio; a unistroke shape recognizer implemented as a pure function in `logic.ts`; vibration feedback only where the browser supports it (not reliable on iPhone), with a click sound as the default.

## Edge cases

- 4–5 players: one team plus spectators who rotate in.
- Operator lifts a finger mid-trace: strokes join.
- Relay has no headphones: turn noise off and ask them to cover their ears (honor system).

## Scope

**MVP:** one team, Grid Tap + Swipe Combo + Dial, rotation, scoring.

**Later:** Trace and Rhythm, two-team mode, a "Telephone" variant where extra players are extra Relays, and a saboteur role.

## Decisions

D-110 and D-111 in [DECISIONS.md](../../docs/DECISIONS.md#signal-chain).

## Open questions

- `difficulty` and `noiseVolume` have no defaults yet. Set them when the spec is expanded before Phase 3.
