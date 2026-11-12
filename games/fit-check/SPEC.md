# Fit Check — SPEC

Status: idea · Slug: `fit-check` · Last updated: 2026-11-12

Working title; rename freely. Built on the shared engine described in `docs/ARCHITECTURE.md` (host-authoritative reducer, pausable game clock, Supabase rooms, content modes, R2 media).

## 1. Pitch

A wardrobe swings open on the TV and the room plays one of two variants. **Dress up:** everyone gets 60 seconds to dress a paper-doll character on their phone for a prompt like "goth" or "big tech CEO", then the room votes on the best fit. **Name the fit:** the wardrobe throws together a random outfit, everyone writes the best descriptor for it, and the room votes on the best one. Both default to head-to-head matchups (Quiplash-style); Battle Royale pits everyone at once.

## 2. At a glance

| Field          | Value                                                                                 |
| -------------- | ------------------------------------------------------------------------------------- |
| Modes          | Party                                                                                 |
| Variants       | Dress up (default), Name the fit                                                      |
| Players        | 3–12 (best at 4–8)                                                                    |
| Session length | Dress up: about 3–4 min per round, 10–15 min for 3 rounds. Name the fit: about 2 min per round |
| Devices        | Host screen; one phone per player                                                     |
| Late join      | Allow: votes right away, plays from the next round                                    |
| Key roles      | None (a disconnected player's last draft is used)                                     |
| Content modes  | Family, General, After dark (prompts and items are both rated)                        |

## 3. How to play

<!-- howto:start -->

Two ways to play. **Dress up:** the wardrobe opens and you get a prompt: goth, your first date, k-pop star. Dress your character to match it before the timer runs out. Switch between a top + bottom or a full-length outfit, swipe through each piece, or open the gallery to see everything you've been dealt. **Name the fit:** the wardrobe throws together a random outfit, and you write the best name for it. Who wears this, and where are they going? Either way, the entries hit the runway and everyone else votes for their favourite. Win the vote, win the points; most points at the end takes the crown.
<!-- howto:end -->

## 4. Roles

| Role        | Device | Sees                                                                                                            |
| ----------- | ------ | --------------------------------------------------------------------------------------------------------------- |
| Dresser     | Phone  | Dress up: their prompt, the dealt wardrobe, a live preview of their character, the timer, a Lock in button       |
| Writer      | Phone  | Name the fit: their matchup's outfit, a text field with a character count, the timer, a Lock in button          |
| Voter       | Phone  | The entries in the current matchup (labelled A/B…), vote buttons; their own entry is greyed out                 |
| Host screen | TV     | Wardrobe intro, who has locked in, the runway, results, awards. In Dress up, never the outfits while dressing   |

Every player is a dresser (Dress up) or a writer (Name the fit) during the play phase, and a voter during `vote` (except in matchups they're part of).

### Variants

- **Dress up:** each matchup gets a prompt and a dealt wardrobe; each player dresses the character for the prompt. Entries are outfits.
- **Name the fit:** each matchup gets one fully random outfit (section 12); each player writes a descriptor for it, up to 60 characters. Entries are descriptors. There are no prompts.
- The variant is a lobby setting and holds for the whole game.

### Matchups

Both variants use the same matchups.

- **1v1 (default):** players are paired at random each round; each pair gets its own prompt (Dress up) or its own outfit (Name the fit). With an odd count of 5 or more, one matchup is a three-way. Everyone not in a matchup votes on it.
- **Battle Royale:** everyone plays the same prompt or describes the same outfit; everyone votes for one entry that isn't their own.
- With exactly 3 players, 1v1 has too few voters, so the lobby switches to Battle Royale and says so.
- Prompts never repeat within a game.

## 5. Flow

| Phase    | Host screen                                                                                                                                                                             | Phones                                                          | Ends when                         | Next                                                                       |
| -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| `lobby`  | Code, players, settings                                                                                                                                                                 | Join, pick avatar                                               | Host starts                       | `intro`                                                                    |
| `intro`  | Wardrobe doors swing open, clothes fan out on the rails, the body outline fades in. Dress up: matchups and prompts appear. Name the fit: each matchup's outfit builds on the body, item by item | Your matchup (prompt or outfit) and opponent; images preload    | ~6 s (host can skip)              | `dress` or `write`                                                         |
| `dress`  | Dress up only. Timer, prompts, who has locked in                                                                                                                                        | Dressing UI (section 9)                                         | All locked in or `dressMs`        | `vote`                                                                     |
| `write`  | Name the fit only. Timer, every matchup's outfit, who has locked in                                                                                                                     | Writing UI (section 9)                                          | All locked in or `writeMs`        | `vote`                                                                     |
| `vote`   | Current matchup. Dress up: prompt + outfits side by side (or a runway grid in Battle Royale). Name the fit: the outfit, with the descriptors as cards beside it                          | Voters tap an entry; players in it see "You're on the runway"   | All eligible votes in or `voteMs` | `result`                                                                   |
| `result` | Vote split, names revealed, points, a camera-flash moment                                                                                                                               | Your points                                                     | 5 s                               | next `vote`, or `scores`                                                   |
| `scores` | Animated leaderboard                                                                                                                                                                    | Your rank                                                       | 5 s                               | `intro` or `awards`                                                        |
| `awards` | One card per matchup played. Dress up: winning outfit, prompt, votes, then a dress-up time-lapse. Name the fit: the outfit builds item by item, then the winning descriptor and votes     | —                                                               | Auto-advance, or host taps        | `final`                                                                    |
| `final`  | Podium + three host buttons: Play again, Change settings, Choose another game                                                                                                           | Your final place                                                | Host choice                       | `intro` (new game, same settings and players), `lobby`, or the gallery `/` |

## 6. Settings

| Key               | Type   | Default    | Range                           | Notes                                                                 |
| ----------------- | ------ | ---------- | ------------------------------- | --------------------------------------------------------------------- |
| `variant`         | enum   | `dress-up` | `dress-up`, `name-the-fit`      |                                                                       |
| `matchups`        | enum   | `1v1`      | `1v1`, `battle-royale`          | 3 players forces `battle-royale`                                      |
| `rounds`          | int    | 3          | 1–8                             |                                                                       |
| `dressMs`         | int    | 60000      | 30000–180000                    | Dress up only                                                         |
| `writeMs`         | int    | 45000      | 20000–90000                     | Name the fit only                                                     |
| `voteMs`          | int    | 15000      | 10000–30000                     |                                                                       |
| `extras`          | multi  | none       | `shoes`, `socks`, `hat`, `face` | Top, bottom and full-length are always on. Applies to both variants  |
| `perSlot`         | int    | 8          | 4–20                            | Dress up only. Items dealt per slot                                   |
| `wildcardShare`   | number | 0.25       | 0–0.5                           | Dress up only. Share of dealt items picked at random instead of by tag |
| `fullWardrobe`    | bool   | false      |                                 | Dress up only. Skip dealing; everyone sees every item                 |
| `promptPick`      | enum   | `random`   | `random`, `host`                | Dress up only. `host` lets the TV pick or reroll each prompt          |
| `anonymousVoting` | bool   | true       |                                 | Names appear only in `result`                                         |

The lobby hides the settings that don't apply to the chosen variant.

## 7. State model

```ts
type Variant = "dress-up" | "name-the-fit";
type Slot = "top" | "bottom" | "full" | "shoes" | "socks" | "hat" | "face";
type WearMode = "separates" | "full";

interface Outfit {
  mode: WearMode; // decides whether top+bottom or full is shown
  items: Partial<Record<Slot, string>>; // item ids; keeps top/bottom AND full so toggling restores choices
}

interface LogEntry {
  t: number; // ms since dress started
  change: { slot: Slot; itemId: string | null } | { mode: WearMode };
}

// Dress up: one per player per round
interface Entry {
  playerId: string;
  outfit: Outfit;
  log: LogEntry[]; // drives the time-lapse replay
  locked: boolean;
  autoDressed: boolean; // true only if no draft ever arrived
}

// Name the fit: one per player per round
interface Caption {
  playerId: string;
  text: string; // trimmed, at most 60 characters; "" if nothing was written
  locked: boolean;
}

interface Matchup {
  id: string;
  playerIds: string[]; // 2 or 3 in 1v1; everyone in battle royale
  promptId?: string; // Dress up only
  dealt?: Partial<Record<Slot, string[]>>; // Dress up only; identical for every player in this matchup
  outfit?: Outfit; // Name the fit only; the random outfit everyone in this matchup describes
  votes: Record<string, string>; // voterId -> playerId voted for
}

interface Round {
  index: number;
  matchups: Matchup[];
  entries: Record<string, Entry>; // Dress up, by playerId
  captions: Record<string, Caption>; // Name the fit, by playerId
}

interface Award {
  roundIndex: number;
  matchupId: string; // the card reads the prompt, outfit or winning descriptor from the matchup
  winnerIds: string[];
  share: number;
}

interface State {
  phase:
    | "lobby"
    | "intro"
    | "dress"
    | "write"
    | "vote"
    | "result"
    | "scores"
    | "awards"
    | "final";
  config: Config;
  roundIndex: number;
  rounds: Round[];
  voteIndex: number; // which matchup is on the runway
  scores: Record<string, number>;
  usedPromptIds: string[]; // Dress up only
  awards: Award[];
  awardIndex: number;
  timers: Timer[]; // engine-managed, game-clock based
}
```

## 8. Intents

| Intent                                     | Sent by     | Valid in phase                                   | Effect                                                                                       |
| ------------------------------------------ | ----------- | ------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `draft { outfit, log }`                    | Dresser     | `dress`                                          | Host stores the latest; phones send at most every 2 s and only when changed                  |
| `lock { outfit, log }`                     | Dresser     | `dress`                                          | Final outfit; locks the entry. Rejected if any item isn't in their dealt set                 |
| `draftCaption { text }`                    | Writer      | `write`                                          | Host stores the latest; phones send at most every 2 s and only when changed                  |
| `lockCaption { text }`                     | Writer      | `write`                                          | Final descriptor; locks the caption. Rejected if blank after trimming or over 60 characters  |
| `vote { matchupId, playerId }`             | Voter       | `vote`                                           | Rejected if the voter is in that matchup, votes for themselves, for a blank caption, or already voted |
| `rerollPrompt { matchupId }`               | Host screen | `intro`, Dress up, when `promptPick` = `host`    | Swaps the prompt and re-deals that matchup                                                   |
| `playAgain` · `changeSettings` · `newGame` | Host screen | `final`                                          | New game with same settings / back to `lobby` / leave to the gallery                         |

Dressing (each swipe or tap) and typing are local to the phone; only drafts and the lock go over the network.

## 9. Views

**Dresser phone (Dress up, during `dress`)**

- Top half: their character preview with the outfit layered on.
- A big segmented toggle: **Separates | Full-length**. Switching hides the other set but keeps its picks, so switching back restores them.
- Slot chips under the preview (Top, Bottom or Full, then any enabled extras). Swiping left/right on the preview changes the item in the selected slot.
- **Gallery** button: a grid of every dealt item for the selected slot; tap to wear.
- Extras start on "None"; core slots start pre-filled with a random dealt item, so no outfit is ever empty.
- Timer bar and **Lock in**.

**Writer phone (Name the fit, during `write`)**

- Top half: their matchup's outfit, rendered with the same layering component.
- A single-line text field with a live character count (60 max) and the shared free-text input.
- Timer bar and **Lock in** (disabled while the field is blank).

**Voter phone (during `vote`):** the matchup's entries as cards labelled A, B (C), tap to vote, then "Vote locked". In Name the fit, blank captions show as "No answer" and can't be tapped.

**Host (during `dress`):** prompts, timer, avatars ticking to "locked in". Outfits stay hidden until `vote`.

**Host (during `write`):** every matchup's outfit, timer, avatars ticking to "locked in". Descriptors stay hidden until `vote`.

Private data: each phone receives only its own matchup's dealt items or outfit, and its own entry or caption. Other players' entries reach phones only in `vote`.

## 10. Timers and pause

| Timer              | Default   | On end                                                          |
| ------------------ | --------- | --------------------------------------------------------------- |
| `intro`            | 6 s       | Go to `dress` or `write`                                        |
| `dress`            | `dressMs` | Use each player's last draft (or starting outfit), go to `vote` |
| `write`            | `writeMs` | Use each player's last draft (blank if none), go to `vote`      |
| `vote`             | `voteMs`  | Count votes cast so far, go to `result`                         |
| `result`, `scores` | 5 s       | Advance                                                         |

Pause freezes all of the above. Phones show the pause overlay and ignore swipes, typing and votes; locked entries stay locked. The wardrobe intro, outfit builds and time-lapse animations pause mid-motion. Resume runs the standard 3-2-1.

## 11. Scoring

Both variants score the same way.

```
points = round(1000 × votesReceived / votesCastInMatchup)
```

- 1v1 sweep (every vote, with at least 2 voters): +250.
- No votes cast: the matchup's 1000 points are split evenly among non-blank entries.
- Battle Royale uses the same formula across the whole room, +250 to the top entry (shared on ties).
- Name the fit: a matchup with fewer than two non-blank captions skips its vote. A lone caption scores 1000 with no sweep bonus; if every caption is blank, nobody scores and no award is given.

Worked example, 1v1 with 3 voters: a 2–1 split gives 667 and 333; a 3–0 sweep gives 1000 + 250 = 1250 and 0.

**Awards:** every matchup played is a category: its prompt in Dress up, its outfit in Name the fit. Its winner is the highest vote share in that matchup (ties share the award). The overall champion is the highest total score.

## 12. Content

### Character and layering

- One 2D base body, front-facing, on a fixed 1024 × 2048 px transparent canvas (`body.png`), plus an outline version (`body-outline.png`) for the intro.
- Every clothing item is drawn **on that same canvas**: you place and scale it over the body in an image editor, hide the body layer, and export the item alone at the full canvas size. No anchor points.
- Default stacking, bottom to top: body → socks → shoes → bottom → top → full → face → hat. An item can override its layer with `z` (for example, wide trousers drawn over shoes).
- A full-length item hides top and bottom. Separates hides full.

### Asset pipeline

1. Find an item image you're allowed to use. Note the source URL and license.
2. Place it on `body.png` in your editor, export the item alone at 1024 × 2048 to `games/fit-check/media/wardrobe/<slot>/<id>.png`.
3. Add an entry to `src/content/wardrobe.json`.
4. `pnpm --filter fit-check wardrobe:build` trims the transparent edges, records each item's box (x, y, w, h on the canvas), writes a WebP and a thumbnail, and outputs `wardrobe.generated.json`. The renderer places each trimmed image back at its box, so you only ever align visually.
5. `pnpm upload-media` publishes everything to the R2 bucket.

```ts
interface WardrobeItem {
  // wardrobe.json, written by hand
  id: string; // 'top-black-mesh'
  slot: Slot;
  name: string; // shown in the gallery
  tags: string[]; // style tags, see below
  rating: "family" | "general" | "after-dark";
  z?: number; // layer override
  source: string; // where the image came from
  license: string; // 'CC0', 'CC BY 4.0', …
  attribution?: string; // required for CC BY
}
// wardrobe.generated.json adds: file, thumb, box: { x, y, w, h }
```

**Licensing:** prefer CC0 / public domain. CC BY is fine with credit; credits are listed on the game page. Skip anything whose license forbids modification or redistribution, and when unsure, leave it out. `validate-content` rejects items without `source` and `license`, and CC BY items without `attribution`.

### Prompts (Dress up)

```ts
interface Prompt {
  id: string;
  text: string;
  tags: string[];
  rating: "family" | "general" | "after-dark";
}
```

Starting tag vocabulary: `dark`, `edgy`, `punk`, `formal`, `glam`, `romantic`, `cute`, `smart-casual`, `casual`, `minimal`, `preppy`, `streetwear`, `sporty`, `techwear`, `retro`, `summer`, `winter`, `costume`.

Seed prompts:

| Prompt                     | Tags                         |
| -------------------------- | ---------------------------- |
| Goth                       | dark, edgy, punk             |
| First day of university    | casual, streetwear, preppy   |
| High school prom           | formal, glam, romantic       |
| Big tech CEO               | minimal, casual, techwear    |
| Design company interviewee | smart-casual, minimal, retro |
| Your first date            | smart-casual, romantic, cute |
| K-pop star                 | glam, streetwear, cute       |
| Giving butterflies         | romantic, cute, glam         |
| Hell is hot                | summer, edgy, costume        |

### Dealing (Dress up, per matchup, seeded)

For each enabled slot:

1. Candidates = items allowed by the room's content mode.
2. Tagged pool = candidates sharing at least one tag with the prompt.
3. Deal `round(perSlot × (1 − wildcardShare))` from the tagged pool and the rest from the remaining candidates (the wildcards). If the tagged pool runs short, top up with random candidates.
4. Shuffle once. Every player in the matchup gets the same list in the same order.

Battle Royale deals one set for the whole room. `fullWardrobe` skips dealing.

### Random outfit (Name the fit, per matchup, seeded)

1. Pick the wear mode at random: separates or full-length, 50/50.
2. For each core slot of that wear mode (top and bottom, or full) and each enabled extra, pick one item at random from that slot's items, through `pickContent` with the room's content mode. Tags are ignored.
3. Every player in the matchup sees the same outfit.

Battle Royale builds one outfit for the whole room. No item is reused across the outfits of one round when the wardrobe has enough items for that slot.

Launch target: about 15 items each for top, bottom and full-length; extras can start at 5–10 each.

### Descriptors

- Free text, trimmed, 1–60 characters, through the shared free-text input.
- In Family mode the host masks profanity when a caption is locked or taken from a draft, so the TV and phones only ever see the masked text ([CONTENT_GUIDE.md](../../docs/CONTENT_GUIDE.md#free-text)).

## 13. Tech dependencies

- **Rendering:** DOM layers. A container with `aspect-ratio: 1 / 2`; each item is an absolutely positioned `<img>` at `left: x/1024`, `top: y/2048`, `width: w/1024` (as percentages). The same component renders on phones, the TV runway, awards and the share page, in both variants.
- **Swipe and gallery:** Pointer Events for swipes on the preview; a simple grid modal for the gallery.
- **Intro:** CSS 3D transforms or Motion for the doors and rails; thumbnails come from R2.
- **Preload:** phones fetch their dealt thumbnails and full images (or their matchup's outfit) during `intro`.
- **Time-lapse:** replays an entry's `log`, sped up so each replay lasts at most 4 s, with a small pop as each item lands. Name the fit's outfit build uses the same pop, one item at a time in stacking order.
- **Shared packages:** the voting module and the free-text input are built here first (Fit Check leads Phase 5) and reused by the later Phase 5 games.
- **Network budget:** drafts (≤1 per 2 s per player) plus one lock and one vote each per round; well within the Realtime limits.

## 14. Persistence and sharing

- Nothing is stored beyond the session except the standard host recovery snapshot.
- Each phone keeps its latest draft (outfit or descriptor) in localStorage so a reconnect mid-round restores it.
- **Share results:** payload = the game's awards (variant, prompt or outfit as item ids, winner display names, winning outfits or descriptors, vote shares). The share page renders the outfits from R2 using the same layering component.

## 15. Edge cases

- **Dresser disconnects mid-dress:** their last draft is used. If none arrived, the starting outfit is used and marked `autoDressed`.
- **Writer disconnects mid-write:** their last draft is used. If none arrived, their caption is blank.
- **Reconnect mid-round:** the phone restores its local draft; the host resends the view with time left.
- **Odd player count (5+) in 1v1:** one three-way matchup.
- **A player leaves between rounds:** re-pair for the next round.
- **Late joiner:** votes in the current round, plays from the next. If that makes the count odd, a three-way matchup is used.
- **Nobody votes:** points split evenly; the award is shared.
- **Blank captions (Name the fit):** shown as "No answer" and can't receive votes; see section 11 for matchups with fewer than two real captions.
- **Locked outfit containing an item outside the dealt set** (stale client): rejected, with a nudge to re-lock.
- **Wardrobe slot empty for the content mode** (Name the fit): that slot is left bare; if a core slot is empty, the other wear mode is used.

## 16. Look and sound

- Fashion-show energy: a wooden wardrobe intro, then a runway with a spotlight; a camera-flash burst and shutter sound on each result.
- Phones feel like a boutique fitting room: big swipe area, satisfying hanger "clack" on each change.
- Name the fit: the outfit assembles on the body with a hanger "clack" per item; descriptors appear on the runway as fashion-magazine captions.
- Awards play as a runway walk: card slides in, the outfit builds, votes count up.
- Wardrobe, body and sounds load from R2 (`games/fit-check/…`).

## 17. Showcase states

- `cover`: wardrobe half open, rails full, body outline in front.
- `vote-1v1`: "Goth" matchup, two outfits side by side, vote bars at 60/40.
- `vote-name-the-fit`: a random outfit with two descriptors beside it, vote bars at 75/25.
- `awards`: a winner card mid-time-lapse.

## 18. Scope

**MVP**

- [ ] One 2D body; top, bottom, full-length slots with layering
- [ ] Asset pipeline (`wardrobe:build`, validator, R2 upload)
- [ ] Tag-based dealing with wildcards; same set per matchup
- [ ] Phone dressing UI: mode toggle, swipe, gallery, lock in
- [ ] Name the fit: seeded random outfits, writing UI, blank-caption handling, Family profanity masking
- [ ] Variant setting in the lobby, hiding settings that don't apply
- [ ] 1v1 and Battle Royale in both variants; 3-player auto switch; three-way matchups
- [ ] Shared voting module and free-text input; scoring, leaderboard
- [ ] Awards slideshow with time-lapse and outfit build; Play again / Change settings / Choose another game
- [ ] Extras (shoes, socks, hat, face) as settings, ready for when assets exist

**Later**

- Players vote on the prompt from 3 options
- More body options (each one needs every item redrawn)
- A Shuffle button on the phone
- "Dress your rival" mode: you style your opponent's character
- Save-the-look image export

## 19. Acceptance tests

- Given 5 players in 1v1, the round has one pair and one three-way matchup, and every player votes on every matchup they're not in. This holds in both variants.
- Given 3 players, the lobby switches to Battle Royale and explains why.
- Both players in a matchup receive identical dealt lists (Dress up) or the identical outfit (Name the fit).
- With `perSlot` 8 and `wildcardShare` 0.25, each slot deals 6 tagged and 2 wildcard items when enough tagged items exist.
- Picking a full-length item hides top and bottom; toggling back to Separates restores the previous top and bottom.
- A Name the fit outfit in separates has a top and a bottom and no full-length item; in full-length it has a full item and no top or bottom; every enabled extra is filled; every item is allowed by the room's content mode.
- The same seed produces the same Name the fit outfits.
- Pausing during `dress` or `write` with 20 s left resumes with 20 s after the 3-2-1; swipes and typing during the pause do nothing.
- A vote for yourself, in your own matchup, or for a blank caption is rejected with a reason.
- A blank or 61-character `lockCaption` is rejected with a reason.
- In Family mode, a profane caption reaches the TV and phones masked.
- A Name the fit matchup where only one caption is non-blank skips its vote and scores that caption 1000.
- A phone that disconnects at 0:45 of dressing or writing gets its last draft entered.
- A 2–1 vote scores 667 / 333; a 3–0 vote scores 1250 / 0.
- Each awards time-lapse lasts at most 4 s and ends on the exact locked outfit.
- In `final`, Play again starts a new game with the same players and settings and no repeated prompts; Change settings returns to the lobby; Choose another game returns to the gallery.

## 20. Decisions and open questions

**Decisions**

- One 2D body.
- Items are hand-placed on the shared body canvas; the build script trims them, so there are no manual anchor points.
- Both players in a matchup get the same dealt clothes.
- The end replay is a winners slideshow with a short dress-up time-lapse per winner.
- Default slots are top, bottom and full-length; extras are opt-in settings.
- 1v1 is the default; Battle Royale is the alternative.
- Two variants, Dress up and Name the fit, both in the MVP ([D-150](../../docs/DECISIONS.md)).
- Name the fit reuses the matchups and gives each matchup one fully random outfit ([D-151](../../docs/DECISIONS.md)).
- Fit Check is the lead game of Phase 5 and builds the shared voting module and free-text input ([D-152](../../docs/DECISIONS.md)).

**Open questions**

- Final name (working title: Fit Check). Name of the reverse variant (working title: Name the fit).
- Should voting stay anonymous until the result? (Default: yes.)
- Should extras turn on by default once enough assets exist for them?
- Should a room be able to mix variants round by round? (Default: no, one variant per game.)
