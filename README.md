# party games 🎈

A local party game platform — think Jackbox, but our own games, and **no host
computer required**. Everyone plays from the web browser on their phone. One
person **creates a room**, gets a code + shareable link, and friends **join**.
A computer or TV can optionally **spectate** — a big-screen scoreboard showing
teams, standings, and the live leaderboard. Built for small groups (up to ~25).

## Play it

`index.html` at the repo root is the whole app, so GitHub Pages serves it at:

**https://9average9.github.io/partygames/**

> Note: right now this is the **single-device preview** — the lobby looks and
> feels real on your phone, but two separate phones don't sync yet. Real
> phone-to-phone play switches on at the server step (see roadmap).

## The model (no host computer)

- **Create** — any phone starts a room, becomes the *host* player (gets the
  Start button), and shares the code/link. The host is still a normal player.
- **Join** — friends enter the code (or open the link) on their phones.
- **Spectate** — optional. A TV/computer/any device opens the spectator screen
  to display the room code, invite link, and the live leaderboard. Purely a
  second perspective — the game runs entirely on phones without it.

## How it's being built

Front end first with a *fake server*, then the **real server** plugs into the
same seam at the end — no UI rewrite. See "The server seam" below.

### Roadmap

- [x] **Step 1 — Lobby.** Create / join / spectate; host hand-off; players
      appear on the spectator scoreboard.
- [x] **Step 2 — First game: Hand Duel.** A spin on rock/paper/scissors — fold
      an on-screen hand to shape your move; team matchmaking, a live bench
      tap-battle, scoring, and a party-night payout.
- [ ] Step 3 — Polish the Hand Duel; more games.
- [ ] Step 4 — Real server (Node + Socket.IO) so separate phones sync live,
      plus the persistent room-level **party-night leaderboard** (two-tier
      scoring: per-game match points vs. per-player night points, across
      shuffling teams and free-for-all games).

## Files

| File | What it is |
|------|------------|
| `index.html` | The lobby app (Pages entry point). Create / join / spectate on one screen. |
| `game.html` | **Hand Duel** — the full stitched loop: versus screen → hand duel with a live bench tap-battle → reveal → scoring → party-night payout. Plays until everyone has dueled twice with rotating matchups. |
| `game-duel.html` | Duel-core prototype (the hand mechanic on its own, vs a bot). |
| `game-versus.html` | The team-vs-team matchmaking intro on its own. |
| `game-tapbattle.html` | The bench tap-battle + scoring on its own. |

Everything is self-contained (no build) and runs **on one device** for now —
each game uses a *fake server* with simulated opponents. Real phone-to-phone
sync arrives at the server step.

## The server seam

All actions go through one `Net` object. The real Socket.IO server will expose
the **same methods**, so plugging it in is a one-object swap:

```
Net.createRoom({name})         -> { ok, code, id }   // you become the host player
Net.joinRoom(code, {name})     -> { ok, id, error }
Net.appointHost(hostId, newId) -> host hands the crown to another player
Net.leaveRoom(playerId)        -> if the host leaves without appointing, the crown
                                  passes to the next player in join order after the
                                  host (wrapping around); an empty room ends
Net.startGame(playerId)        -> host only; starts the game
Net.linkFor(code)              -> shareable invite link
Net.subscribe(fn)              -> fn(room) whenever the room changes
```

Room shape (also the future server's shape):

```
{ code, phase: 'lobby' | 'playing', hostId, players: [ { id, name, avatar, color, score } ] }
```

Avatars are the player's **initials** on a colored disc; UI icons are inlined
[Lucide](https://lucide.dev) SVGs (MIT). The theme is **light by default** and
adapts to the viewer's dark mode.
