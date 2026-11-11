# Game Night — More Game Ideas

Candidates after the first five, chosen so each one reuses something those games already build: the relative-error scorer, the audio stack, the drawing canvas, private roles, or voting. Nothing here is locked. A game moves out of this list when it gets a `games/<slug>/SPEC.md`.

Effort is S (a weekend), M (a few weekends) or L.

| Game             | Mode   | Pitch                                                                                                                                     | Builds on                                                  | Effort |
| ---------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------ |
| Ballpark         | Party  | Estimation trivia: "How many steps up the Eiffel Tower?" Same relative scoring as Gap Years.                                              | Gap Years scoring + reveal                                 | S      |
| Map Drop         | Party  | Drop a pin where something happened; points by distance relative to the size of the region.                                               | Gap Years scoring, events data + lat/lng, an SVG world map | M      |
| Dial In          | Party  | One player sees a hidden target on a spectrum ("overrated ↔ underrated") and gives a one-word clue; everyone turns a dial on their phone. | Signal Chain's Dial control, private views                 | S      |
| Odd One Out      | Party  | Everyone gets the same secret word except one player, who only gets the category. One clue each, then vote out the impostor.              | Private views, voting                                      | S      |
| Sketch Telephone | Party  | Write a prompt, the next player draws it, the next guesses the drawing, and so on; chains are revealed at the end.                        | Signal Chain's drawing canvas                              | M      |
| Bluff Fill       | Party  | A strange true fact with a blank. Everyone writes a fake answer, then votes for the real one; points for fooling friends.                 | Voting, verified content pipeline                          | M      |
| Forehead         | Screen | Phone on your forehead; friends shout clues; tilt down for correct, up to pass.                                                           | Content lists, device-orientation sensor                   | S      |
| Hot Seat Debate  | Hybrid | Two players argue absurd opposite positions for 45 s each while the audience fires interruption cards; audience votes the winner.         | Slide Roulette's timers, reactions and ratings             | S      |
| Foley Night      | Party  | A silent scene plays on the TV; everyone has a soundboard on their phone and adds live sound effects; replay the result.                  | Jam audio pipeline, Pexels video search                    | M      |
| Majority Rules   | Party  | Quick-fire questions ("Best pizza topping?"); score by matching what most people said.                                                    | Text input, fuzzy matching, voting                         | S      |
| Who Said It      | Party  | Everyone answers a personal prompt anonymously; guess who wrote each answer.                                                              | Text input, voting                                         | S      |

**Suggested next three (Phase 5):** Bluff Fill, Odd One Out (the best pure social game here, and tiny), and Ballpark (nearly free once Gap Years exists).

## Shared systems worth building once

Several games want these, so each is built once in a shared package the first time a game needs it:

- **Voting module:** Odd One Out, Bluff Fill, Who Said It, Hot Seat Debate, Slide Roulette ratings.
- **Free-text input** with profanity masking and fuzzy matching: Bluff Fill, Majority Rules, Who Said It.
- **Drawing canvas with stroke streaming:** Signal Chain, Sketch Telephone.
- **Relative-error scorer:** Gap Years, Ballpark, Map Drop.
