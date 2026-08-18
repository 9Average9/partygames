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
- [ ] Step 2 — First game (prompt → answers → vote → winner + scoring).
- [ ] Step 3 — Teams & more games.
- [ ] Step 4 — Real server (Node + Socket.IO) so separate phones sync live.

## Files

| File | What it is |
|------|------------|
| `index.html` | The whole app — one self-contained file, no build. Open it in any browser (great on a phone), or visit the Pages URL above. Shows the optional spectator screen + a phone with Create / Join / Spectate, all on one screen so you can test solo. |

Because there's no server yet, everything runs **on one device**: create a room,
then use "Add a test player" to watch the scoreboard fill. Real phone-to-phone
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
