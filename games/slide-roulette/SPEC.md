# Slide Roulette — SPEC

Status: idea · Slug: `slide-roulette` · Last updated: 2026-11-04 · Condensed; expand to the full [template](../../docs/GAME_SPEC_TEMPLATE.md) before Phase 2.

## Pitch

Presentation improv. One player presents a deck they've never seen while another player picks each next slide on their phone. A new slide appears every 10 seconds, alternating photo and text, and the presenter has to make it make sense.

## At a glance

| Field          | Value                                                                          |
| -------------- | ------------------------------------------------------------------------------ |
| Modes          | Party; Hybrid fallback with random picks and no phones                         |
| Players        | 2–12; best at 4–8                                                              |
| Session length | About 2.5 min per talk; 15–25 min for a full round                             |
| Devices        | Host screen; a phone for every player (the presenter's shows what's coming up) |
| Late join      | Allow (spliced into the pairing loop, see below)                               |
| Key roles      | Presenter and Slide Picker (a disconnect auto-pauses)                          |
| Content modes  | Family, General, After dark                                                    |

## How to play

<!-- howto:start -->

You get a talk title and a deck you've never seen. A new slide appears every 10 seconds, alternating between a photo and a line of text, and you present it like you meant it. Your phone shows what's coming up next, and once per talk you can hit Skip to swap the current slide for a blank one and wing it. Meanwhile your Slide Picker chooses each next slide from three options on their phone. Everyone else reacts and rates the talk. In one round, everyone presents once and picks once.

<!-- howto:end -->

## Pairing: one loop

At the start of a round the host shuffles all players into a single loop, P1 → P2 → … → Pn → back to P1. In talk k, Pk presents and the next player around the loop picks for them.

- Everyone presents once and picks once.
- With 3 or more players nobody picks for each other, and the group never splits into separate pairs or subgroups.
- With exactly 2 players they pick for each other; that can't be avoided.
- A late joiner is spliced in between two players who haven't presented yet. If there's no such spot, they watch until the next round.
- A player who leaves is removed and the loop closes the gap: the next player around now picks for the one before.
- If `rounds` is less than everyone, the first k presenters in loop order go.

## Roles

| Role         | Device | Sees                                                                                                                                                                                                                |
| ------------ | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Presenter    | Phone  | **Up next**: the slide the Picker has locked in (or "Picker is choosing…"); seconds left on this slide; slides left; a Skip button (1 per talk)                                                                     |
| Slide Picker | Phone  | Current slide thumbnail, countdown, and 3 option cards showing the raw content (the photo, or the line of text). After a tap, a preview of that option as a slide; they can change their pick until the slide flips |
| Audience     | Phone  | Reaction buttons during the talk, a rating form after                                                                                                                                                               |

The TV only ever shows the current slide: never the options and never what's coming up.

## Flow

| Phase      | Host screen                                               | Phones                                                                       | Ends when                                                  | Next                  |
| ---------- | --------------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------- | --------------------- |
| `lobby`    | Code, players, deck theme, content mode                   | Join, pick avatar                                                            | Host starts                                                | `assign`              |
| `assign`   | "Presenter: Sam · Picker: Ana"                            | Each phone shows its role for this talk                                      | 4 s                                                        | `title`               |
| `title`    | Generated talk title                                      | Picker gets the first options (photos); presenter sees the title and up next | `titleMs`                                                  | `slide`               |
| `slide` ×N | Current slide + thin progress bar; floating reactions     | Picker chooses; presenter sees up next + Skip; audience reacts               | `slideMs`; if nothing picked, a random option from the set | `slide` or `outro`    |
| `outro`    | "Thank you! Questions?"                                   | —                                                                            | 8 s                                                        | `rate`                |
| `rate`     | "Rate the talk", live tally                               | Audience: 1–5 stars on Commitment and Chaos                                  | All voted or 20 s                                          | `assign` or `results` |
| `results`  | Podium: best presenter, best picker (most-reacted slides) | Your placing                                                                 | Host                                                       | `lobby`               |

## Slide rotation

After the title slide, slides alternate: photo, text, photo, text… All three options in a set are the type of the slide they'll fill.

| Type  | Options come from                                                            | Example                                                                                                      |
| ----- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Photo | Theme photos and wildcard photos                                             | Fireworks (theme: celebration); a llama in sunglasses (wildcard)                                             |
| Text  | `text/transitions.json`, `confessions.json`, `teasers.json`, `closings.json` | "BUT THEN..." · "I was NOT prepared for this" · "You wouldn't guess what happened next" · "In conclusion..." |
| Blank | Only from the presenter's Skip                                               | The skin's empty background with a soft spotlight vignette                                                   |

Later, a Chart type (generated SVG with absurd labels) joins the rotation as photo → text → chart, and Callback lines ("Remember the fireworks?") join the text pool.

## Option rules

1. Three options per set, all the same type as the coming slide.
2. Photo sets are usually 2 theme photos + 1 wildcard. If the photo slides left equal the theme photos still needed to reach `minThemeImages`, all three are theme photos.
3. Text sets draw from three different text files when possible; closings appear only in the last two text slides.
4. No photo or line repeats within a game. Everything is filtered by the room's content mode (see [CONTENT_GUIDE.md](../../docs/CONTENT_GUIDE.md)).
5. The talk title comes from templates such as "{Topic}: What {Group} Won't Tell You", with the topic drawn from the theme.

## Skip

Once per talk (setting `presenterSkips`), the presenter taps Skip during a slide. The current slide is replaced by the blank slide for its remaining seconds; the slide timer doesn't change and the deck length doesn't change. The Picker carries on as normal, so the next slide is still theirs.

## Settings

| Key                 | Type | Default  | Range                                                                  |
| ------------------- | ---- | -------- | ---------------------------------------------------------------------- |
| `slideCount`        | int  | 10       | 5–20                                                                   |
| `slideMs`           | int  | 10000    | 5000–20000                                                             |
| `optionsPerPick`    | int  | 3        | 2–4                                                                    |
| `minThemeImages`    | int  | 2        | 0–5                                                                    |
| `presenterSkips`    | int  | 1        | 0–3                                                                    |
| `theme`             | enum | random   | celebration, travel, office, nature, food, sports, space, pets, random |
| `skin`              | enum | random   | corporate, wordart, startup, science-fair                              |
| `audienceReactions` | bool | true     | —                                                                      |
| `rounds`            | enum | everyone | everyone, or 1–12                                                      |
| `onPickerLost`      | enum | pause    | pause, auto-pick                                                       |

## Intents

| Intent                       | Sent by   | Valid in phase          | Effect                                                                         |
| ---------------------------- | --------- | ----------------------- | ------------------------------------------------------------------------------ |
| `pick { optionId }`          | Picker    | `title`, `slide`        | Sets or changes the next slide; the presenter's up next updates                |
| `skip`                       | Presenter | `slide`, skips left > 0 | Current slide becomes the blank slide for its remaining time                   |
| `react { emoji }`            | Audience  | `slide`                 | Floating reaction on the TV; capped at 2 per second per phone, sent in batches |
| `rate { commitment, chaos }` | Audience  | `rate`                  | 1–5 stars each                                                                 |

## Views

- **Presenter:** current slide, up next (once picked), seconds left, slides left, skips left.
- **Picker:** current slide, the option set with preview data, their current pick.
- **Audience:** current slide and reaction buttons, nothing upcoming.
- **Host:** current slide only.

## Photos

- MVP: a build-time pool. `pnpm build:images` queries Pexels for each term in `themes.json`, keeps about 40 landscape photos per term, and writes their URLs, alt text, average color and photographer credit to `content/photo-pool.json`. Runtime makes zero API calls. Re-run monthly.
- Every search term is tagged `people: true | false` ("office meeting" vs "golden retriever"), and each pooled photo inherits it.
- After dark mode uses only `people: false` photos, plus AI-generated people images stored in the R2 bucket under `art/ai/slide-roulette/` and listed in `content/ai-pool.json` with a rating (see [CONTENT_GUIDE.md](../../docs/CONTENT_GUIDE.md#photos)).
- Pexels' defaults are 200 requests per hour and 20,000 per month ([Pexels API docs](https://www.pexels.com/api/documentation/)). Each Pexels slide shows a small "Photo: Name on Pexels" credit; AI images show "AI-generated". The game page and end screen carry the Pexels link.
- The host preloads and decodes the next options' images during the current slide, using `avg_color` as the placeholder. A photo that fails to load is swapped for another photo from the same set's pool.
- Safety: only curated search terms, a blocklist of photo ids, and a "skip this one" button on the picker's phone that replaces an option.
- Later: a "Fresh photos" toggle that calls `/api/images`, cached 24 h per search term.

## Timers and pause

`title` and `slide` timers freeze; the picker's choice stays locked in; Skip and reactions are disabled while paused. After the 3-2-1, the current slide (or blank slide) gets its remaining time.

## Scoring (light)

Presenter score = average stars × 100. Picker score = reactions received during slides they chose.

## Edge cases

- Picker disconnects: `onPickerLost` decides (pause by default, or auto-pick).
- Presenter's phone disconnects: pause; the host can tap "Continue without presenter phone", which hides up next and Skip for that talk.
- Only 2 players: no audience and no ratings; they pick for each other.
- No phones at all: Hybrid mode deals a fully random deck; the host taps to start each talk.

## Look

A parody of office slideware. Four skins (corporate blue, retro WordArt, startup pitch, science-fair poster), cheesy transitions, a clicker sound on each slide change.

## Scope

**MVP:** title + 10 slides alternating photo and text, 3 same-type options with previews, presenter phone with up next and 1 Skip, loop pairing, theme and wildcard photos, 4 text files, reactions, content modes.

**Later:** Chart slides in the rotation, callbacks, ratings and podium, an "interrupting audience question" card, user-made decks.

## Decisions

D-100 to D-103 in [DECISIONS.md](../../docs/DECISIONS.md#slide-roulette).

## Open questions

- The flow table includes `rate` and `results` (ratings and podium), but the scope lists ratings and podium as Later. Decide which before Phase 2.
