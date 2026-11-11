# Game spec template

Every game gets a `games/<slug>/SPEC.md` in this shape, so a person or an AI agent can build the game from its spec plus [ARCHITECTURE.md](ARCHITECTURE.md) alone. `pnpm new-game` copies everything below the line.

A spec may start condensed (pitch, at a glance, how to play, flow, settings, scoring, MVP). It is expanded to every section below before its game's implementation phase starts (see [D-005](DECISIONS.md)).

---

```markdown
# <Game name> — SPEC

Status: idea | prototype | playable · Slug: <slug> · Last updated: YYYY-MM-DD

## 1. Pitch

One or two sentences. What makes it funny, tense or satisfying?

## 2. At a glance

| Field          | Value                                        |
| -------------- | -------------------------------------------- |
| Modes          | screen / party / hybrid                      |
| Players        | min–max (best at N)                          |
| Session length | ~N min                                       |
| Devices        | host screen, phones, mic, speakers           |
| Late join      | allow / spectate / deny                      |
| Key roles      | roles whose disconnect auto-pauses the game  |
| Content modes  | which of family / general / after dark apply |

## 3. How to play

<!-- howto:start -->

Player-facing rules, 120 words max, written to "you". Shown on the game page and in the pause menu.

<!-- howto:end -->

## 4. Roles

Who does what, and what each role can see.

## 5. Flow

| Phase | Host screen | Phones | Ends when | Next |
| ----- | ----------- | ------ | --------- | ---- |

## 6. Settings

| Key | Type | Default | Range | Notes |
| --- | ---- | ------- | ----- | ----- |

## 7. State model

The State interface in TypeScript, with a comment on every field.

## 8. Intents

| Intent | Sent by | Valid in phase | Effect |
| ------ | ------- | -------------- | ------ |

## 9. Views

What the host view and each player view contain. Mark anything private.

## 10. Timers and pause

Each timer: id, duration, what happens when it ends.
What stops on pause (audio, speech, video, animation). Anything special on resume.

## 11. Scoring

Formula with a worked example, tie-breaks, end-of-game ranking.

## 12. Content

Data files and schema (every entry carries a rating), target size, where it comes from,
how it's verified, how each content mode changes what's dealt.

## 13. Tech dependencies

Libraries, APIs, browser features, and the fallback when a feature is missing.

## 14. Persistence and sharing

What's saved, where (memory, localStorage, R2, database), and when.
If the game supports Share results: the payload and what the share page shows.

## 15. Edge cases

Disconnects, too few players, ties, empty submissions, slow networks, refresh mid-round.

## 16. Look and sound

Art direction, palette, fonts, motion, sound cues; media files published to R2.

## 17. Showcase states

Seeded moments used for gallery screenshots.

## 18. Scope

MVP checklist, then later ideas.

## 19. Acceptance tests

Given / when / then checks that prove it works, including a pause mid-timer.

## 20. Decisions and open questions
```
