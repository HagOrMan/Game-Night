# Game Night — Rooms and Protocol

How the host screen and phones find each other and talk. Locked: changes go through [DECISIONS.md](DECISIONS.md). Any change to the message shapes below bumps the protocol version `v`.

A room is just a channel name. The host owns the state, each phone gets only its own view on a private channel, and only views that changed are sent.

## Room codes

- 4 letters, consonants only (no vowels means no accidental words), minus Z, which is easily misread as 2. That leaves 20 letters: 20⁴ = 160,000 codes.
- To claim a code, the host joins its public channel and checks presence for 1 s; if another host is there, it picks a new code.
- The lobby shows the code large, plus a QR code for `https://<site>/j/ABCD`.

## Channels

| Channel              | Who joins             | Carries                                             |
| -------------------- | --------------------- | --------------------------------------------------- |
| `gn:ABCD`            | Host + all phones     | Presence only: who's here, their role, name, avatar |
| `gn:ABCD:<playerId>` | Host + that one phone | Intents up; that player's view, acks and kicks down |

Per-player channels mean a phone never receives other players' intents (no wasted deliveries against the message budget) and private information, like a secret role, never reaches the wrong phone.

## Join, reconnect, leave

1. Phone opens `/j/ABCD`, picks a name and avatar (remembered for next time), and joins `gn:ABCD`.
2. If no presence entry with role `host` appears within 3 s, the phone shows "Room not found".
3. The host sees the new presence entry and joins that player's private channel. The phone repeats `hello` every second until it gets `welcome` and its first view.
4. **Reconnect:** the phone reuses its stored player id, so the host marks it online again and resends its view. No progress is lost.
5. **Late join:** each game declares `lateJoin: 'allow' | 'spectate' | 'deny'`.
6. **Host drops:** phones see the host leave presence and show "Waiting for host". The host reloads, restores from localStorage with the same code, and rejoins the channels.
7. The host can kick a player from the player list.

## Message envelope

```ts
type ToHost =
  | { t: "hello"; v: 1; player: { id: string; name: string; avatar: string } }
  | { t: "intent"; id: string; intent: unknown } // id lets the host dedupe retries
  | { t: "pause" }
  | { t: "resume" };

type ToPlayer =
  | { t: "welcome"; v: 1; game: string; you: string }
  | {
      t: "view";
      seq: number;
      phase: string;
      status: "lobby" | "running" | "paused" | "countdown" | "ended";
      remainingMs?: number;
      view: unknown;
    } // full view, never a diff
  | { t: "ack"; id: string; ok: boolean; reason?: string }
  | { t: "kicked" };
```

- Full views, not diffs: they're small (target under 16 KB) and self-heal after a dropped message. Phones ignore any `seq` lower than the last one seen.
- The host hashes each player's view and sends only when it changed.
- `v` is a protocol version. A phone running an old cached build gets a "Refresh to play" prompt instead of breaking.
- Both envelopes have Zod schemas in `packages/realtime`; every incoming message is parsed before use.
- Trust model: the public key and channel names are guessable, so a determined friend could snoop. Fine for friends; Supabase's private channels with row-level security are the upgrade path if ever needed.

## Message budget

Supabase's free plan allows 100 messages per second across the whole project, counting each message sent _and_ each delivery (see [ARCHITECTURE.md](ARCHITECTURE.md#supabase-free-tier-limits-that-shape-the-design)).

- A host state update to 8 phones costs 9 events (1 sent + 8 delivered). At one update per second that's 9 events/s, fine.
- Views are sent on change, never on a tick. Phones count timers down locally from `remainingMs`.
- High-rate input is batched: Slide Roulette reactions (at most 2/s per phone, batched), Signal Chain Operator input (at most 5 messages/s).
- Jam Room is the exception: 6 players sending 8 notes/s through a relay is about 96 events/s, at the cap. Jam Room therefore uses peer-to-peer WebRTC data channels, with Supabase only for the handshake (see [its spec](../games/jam-room/SPEC.md)).
