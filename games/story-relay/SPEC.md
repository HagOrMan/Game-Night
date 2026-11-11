# Story Relay — SPEC

Status: idea · Slug: `story-relay` · Last updated: 2026-11-04 · Condensed; expand to the full [template](../../docs/GAME_SPEC_TEMPLATE.md) before Phase 1.

## Pitch

Build a story together. One screen deals a premise and a cast, then walks the group through a five-beat story arc, giving each storyteller 30 seconds and a nudge for their part. The screen is there for inspiration and never assumes anything was written down: the mic is optional, and the ending works just as well with no transcript at all.

## At a glance

| Field          | Value                                                           |
| -------------- | --------------------------------------------------------------- |
| Modes          | Screen (one device); Hybrid later, with phones voting on twists |
| Players        | 2–10                                                            |
| Session length | 5–12 min                                                        |
| Devices        | One screen; microphone optional                                 |
| Late join      | Allow (add a name between turns)                                |
| Key roles      | None                                                            |
| Content modes  | Family, General, After dark                                     |

## How to play

<!-- howto:start -->

Pick a premise (or write your own) and meet your cast. Then take turns telling the story. When it's your turn, the screen tells you which part of the story you're on, like "the conflict" or "the climax", and gives you a spark to work in. You get 30 seconds; when time's up, the next person picks up exactly where you stopped. At the end you get the shape of the story you told. If nobody used the mic, it's just a fun memory of how it happened.

<!-- howto:end -->

## Flow

| Phase     | Screen                                                                                                                                     | Ends when                  | Next                  |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------- | --------------------- |
| `players` | Type names (remembered next time)                                                                                                          | Tap "Next"                 | `premise`             |
| `premise` | 3 premise cards; reroll (2 left); "Write your own"                                                                                         | Card chosen                | `cast`                |
| `cast`    | 2–4 characters, each a name + trait + want; reroll or edit any; optional setting and genre; mic toggle (only if supported, off by default) | Tap "Begin"                | `handoff`             |
| `handoff` | "Up next: Maya" + 3-2-1                                                                                                                    | 3 s                        | `turn`                |
| `turn`    | Beat label, beat prompt, spark, timer ring; live transcript only if the mic is on                                                          | `turnMs`, or "Done" tapped | `handoff` or `ending` |
| `ending`  | Generated title, then the storybook (below) and a "Narrate it" button                                                                      | Tap "Finish"               | `saved`               |
| `saved`   | Save on this device · Play again (· Share results, later)                                                                                  | —                          | `premise`             |

## Story arc

Turns = players × `laps` (at least 5). Turns are split across beats by weight, with every beat getting at least one turn:

| Beat              | Share of turns | Example prompt                                           |
| ----------------- | -------------- | -------------------------------------------------------- |
| Setup             | 15%            | "Introduce {hero} in {setting}. What's normal for them?" |
| Inciting incident | 15%            | "Something arrives that {hero} can't ignore."            |
| Rising action     | 35%            | "It gets worse. {rival} makes a move."                   |
| Climax            | 20%            | "Everything comes to a head. Who risks what?"            |
| Resolution        | 15%            | "How has {hero} changed? End it."                        |

Settings let you swap to three-act structure or "Chaos" (no beats, sparks only).

## The ending never needs text

The storybook is built from what the game already knows:

- Title page: generated title, premise, setting and genre.
- Cast page: each character's name, trait and want.
- One page per turn: the teller's name, the beat, the prompt they got and their spark. If that turn has a transcript, it appears under them as an extra.
- "Narrate it" reads a recap aloud with the browser's built-in text-to-speech (`speechSynthesis`), filled from templates: "Our story began with {premise}. Maya set the scene in {setting}. Then Jo brought trouble: {spark}..." When transcripts exist, it reads those instead.

## Content files

Static JSON, at least 100 entries each at launch, every entry carrying a content `rating` (see [CONTENT_GUIDE.md](../../docs/CONTENT_GUIDE.md)):

- `premises.json` ("A lighthouse keeper finds a door in the sea")
- `names.json` (mixed real-world and fantasy names)
- `traits.json` ("terrified of birds")
- `wants.json` ("to finally win the bake-off")
- `sparks.json` (objects, twists such as "someone lies", constraints such as "no one may say no")
- `settings.json`
- beat prompt templates that fill in `{hero}`, `{rival}` and `{setting}`

## Settings

| Key | Default | Range |
| --- | ------- | ----- |
| `turnMs` | 30000 | 15000–90000 |
| `laps` | 1 | 1–3 |
| `structure` | five-beat | five-beat, three-act, chaos |
| `genre` | any | any, fantasy, sci-fi, mystery, rom-com, spooky |
| `characters` | 3 | 2–4 |
| `sparks` | on | on, off |
| `mic` | off | toggle shown only when supported |
| `language` | en-US | |

## Optional speech-to-text, free in the browser

| Browser                         | Live transcription                                       | Notes                                                                                                          |
| ------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Chrome (desktop, Android)       | Yes, partial support                                     | Audio goes to a speech service by default; on-device mode can be requested where a language pack is installed |
| Safari (macOS 14.1+, iOS 14.5+) | Yes, partial support, prefixed `webkitSpeechRecognition` | Needs Siri enabled; not in home-screen web apps                                                                |
| Edge                            | Listed as not supported                                  | Hide the mic toggle                                                                                            |
| Firefox                         | Disabled by default                                      | Hide the mic toggle                                                                                            |

Sources: [caniuse: Speech Recognition](https://caniuse.com/speech-recognition), [MDN: on-device availability](https://developer.mozilla.org/docs/Web/API/SpeechRecognition/available_static).

Implementation notes:

- Feature-detect `SpeechRecognition || webkitSpeechRecognition`; set `continuous` and `interimResults` to true; show interim words faded and final words solid.
- Recognition stops by itself after silence on some browsers, so restart it in `onend` while a turn is running.
- On pause, stop recognition; restart it after the countdown. Each turn's final text is saved to that turn.
- When the mic is turned on, show a one-line notice that in Chrome speech may be processed by an online service unless on-device mode is available.
- Transcripts are editable on the ending screen.
- Later fallback: in-browser Whisper via transformers.js on WebGPU, for browsers without speech recognition (a large one-time model download).

## Timers and pause

`turnMs` freezes and the mic (if on) stops; resuming runs the 3-2-1 and restarts it, so the teller's remaining seconds are intact.

## Persistence

Finished stories (title, premise, cast, and each turn's teller, beat, prompt and spark, plus transcripts when there are any) are saved in localStorage, last 20 kept. Later, "Share results" sends the same outline to a short-lived link (see [ARCHITECTURE.md](../../docs/ARCHITECTURE.md#share-results-later-not-mvp)).

## Edge cases

- A player freezes: "Pass" moves to the next teller and gives them +10 s.
- Mic permission denied: carry on silently; nothing depends on it.
- Very long transcripts are trimmed to the last 600 characters per turn on screen; the full text is kept.

## Scope

**MVP:** names, premise pick and reroll, cast, five-beat arc, timer, the text-free storybook ending with narration, optional mic transcripts in Chrome and Safari.

**Later:** Share results, phones vote on the next spark (Hybrid), secret objectives per player ("work in the word 'banana'"), an optional AI polish of transcribed stories, printable storybook export.

## Decisions

D-140 and D-141 in [DECISIONS.md](../../docs/DECISIONS.md#story-relay).
