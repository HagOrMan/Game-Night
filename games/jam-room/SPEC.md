# Jam Room — SPEC

Status: idea · Slug: `jam-room` · Last updated: 2026-11-04 · Condensed; expand to the full [template](../../docs/GAME_SPEC_TEMPLATE.md) before Phase 4.

## Pitch

Everyone picks an instrument on their phone and plays together through the host's speakers. The host keeps the beat and snaps every note to the grid, which hides network lag and makes beginners sound tight. Modes go from tapping preset chords to playing free notes.

## At a glance

| Field          | Value                                                                                        |
| -------------- | -------------------------------------------------------------------------------------------- |
| Modes          | Party (phones as instruments); Screen (the host's computer keyboard plays, for solo jamming) |
| Players        | 1–8; best at 3–5                                                                             |
| Session length | Open-ended; jams of 2–5 min                                                                  |
| Devices        | Host screen with good speakers; phones                                                       |
| Late join      | Allow, any time                                                                              |
| Key roles      | None                                                                                         |
| Content modes  | None (deals no content; picker hidden)                                                       |

## How to play

<!-- howto:start -->

Pick an instrument on your phone. The big screen shows the beat and the chord the band is on. In Chords mode just tap the glowing chord; in Scale mode every key fits the song, so nothing sounds wrong; Free mode is the full keyboard. Drums keep everyone together. Hit record on the host if something good happens.

<!-- howto:end -->

## Where the sound comes from

| Setting `soundOn` | How it works                                                                                | Trade-off                                                                          |
| ----------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `host` (default)  | Phones send note events; the host plays them through Tone.js, snapped to the next grid step | One coherent mix on good speakers; the press-to-sound delay is up to one grid step |
| `phone`           | Each phone plays its own notes instantly; the host only visualizes                          | Zero delay for the player; tinny phone speakers, phones drift apart                |
| `both`            | Phone plays quietly as a monitor, host plays the mix                                        | Best feel; can sound doubled in small rooms                                        |

## Transport

Notes are the one high-rate input on the platform, so this game uses `WebRTCTransport`: a peer-to-peer data channel from each phone to the host, set to unordered with no retransmits (a late note is useless), with Supabase only carrying the connection handshake. Fallback when peer-to-peer fails (some networks block it): Supabase Broadcast with notes batched every 50 ms and quantize forced to at least 8th notes, keeping a 6-player band under the 100 messages/s budget (see [PROTOCOL.md](../../docs/PROTOCOL.md#message-budget)).

## Musical setup (host lobby)

| Setting        | Default   | Options                                                         |
| -------------- | --------- | --------------------------------------------------------------- |
| `key`          | C         | All 12                                                          |
| `scale`        | major     | major, natural minor, major pentatonic, minor pentatonic, blues |
| `bpm`          | 96        | 60–160                                                          |
| `progression`  | I–V–vi–IV | vi–IV–I–V, i–VI–III–VII, ii–V–I, 12-bar blues                   |
| `barsPerChord` | 1         | 1, 2, 4                                                         |
| `quantize`     | 16th      | off, 8th, 16th                                                  |
| `backing`      | drums     | off, metronome, drums, drums + bass                             |

## Instruments and play modes

| Instrument    | Chords mode                                                                   | Scale mode                                   | Free mode                          |
| ------------- | ----------------------------------------------------------------------------- | -------------------------------------------- | ---------------------------------- |
| Keys          | 4 big chord buttons from the progression; current chord glows; swipe to strum | 2 octaves of in-scale keys; chord tones glow | Chromatic, 2 octaves, octave shift |
| Bass          | Root-note buttons per chord                                                   | In-scale keys, low octave                    | Chromatic                          |
| Lead synth    | —                                                                             | In-scale keys; slide finger to bend          | Chromatic + bend                   |
| Pad / strings | Hold a chord to swell                                                         | In-scale keys                                | Chromatic                          |
| Drums         | 6 pads: kick, snare, hat, clap, tom, crash                                    | same                                         | same                               |
| Theremin      | —                                                                             | Vertical slider snaps to scale               | Continuous pitch                   |

Sounds come from Tone.js synths for the MVP, so there are no sample licenses to track. Sampled instruments come later, and only from sets whose license is checked first, hosted in the R2 bucket.

## Intents

```ts
type JamEvent =
  | { k: "on"; note: number; vel: number; dt: number } // dt = ms since batch start, keeps spacing inside a batch
  | { k: "off"; note: number; dt: number }
  | { k: "chord"; degree: number; strum?: "down" | "up"; dt: number }
  | {
      k: "pad";
      pad: "kick" | "snare" | "hat" | "clap" | "tom" | "crash";
      vel: number;
      dt: number;
    };

type JamIntent =
  | { k: "batch"; events: JamEvent[] }
  | { k: "instrument"; id: string; mode: "chords" | "scale" | "free" };
```

The host schedules each event at `Transport.nextSubdivision(quantize)`. Any note held for more than 4 s without an `off` is released, so a dropped message can't leave a drone. Each player gets at most 8 voices.

## Host screen

One lane per player (avatar, color, instrument icon) with notes as blocks scrolling right to left; the current and next chord shown large; a pulse on every beat; a record button.

## Timers and pause

This game has no game timer; the clock is Tone.Transport. Pause stops the transport and releases every held note. Resume runs the standard 3-2-1, with the numbers landing on beats and click sounds, then restarts the transport on the next bar's downbeat.

## Recording

Tone.Recorder captures the master output to a downloadable audio file. The note events are also saved as compact JSON so a jam can be replayed or re-rendered later. The last 3 jams are kept in IndexedDB.

## Tech

Tone.js; audio unlock on first tap (`useAudioUnlock`); on iPhone, setting `navigator.audioSession.type = 'playback'` stops the ring/silent switch from muting Web Audio on iOS 17+ ([WebKit bug 251532](https://bugs.webkit.org/show_bug.cgi?id=251532)); `latencyHint: 'interactive'`; WebRTC data channels. Computer-keyboard input in Screen mode maps two rows of letters to a scale.

## Edge cases

- A phone disconnects: release its notes, keep its lane greyed.
- Host tab hidden: auto-pause, since browsers throttle background audio timing.
- Phone on silent: show "Your phone's on silent" only when `soundOn` includes the phone.

## Scope

**MVP:** host sound, Supabase batching, Keys (chords), Bass (scale), Drums, 3 progressions, lanes visual.

**Later:** WebRTC, Free mode, Lead and Theremin, recording, phone sound modes, a per-player looper, "song form" with verse and chorus sections.

## Decisions

D-120 and D-121 in [DECISIONS.md](../../docs/DECISIONS.md#jam-room).

## Open questions

- The MVP list puts WebRTC in Later, but Phase 4's deliverables and its gate ("4 phones jam for 5 min with no dropouts") include WebRTCTransport. Decide before Phase 4 whether WebRTC is in this game's MVP.
