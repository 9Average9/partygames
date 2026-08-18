# SHINDIG 🎉 (working name)

A local, gather-around-the-TV party game platform — think Jackbox, but our own
games. One **big screen** (a laptop/TV) shows the shared game; everyone else
plays from the **web browser on their phone**. No app installs, no accounts —
you join a room with a short code. Built for small groups (up to ~25 players).

## How it's being built

We're building the **front end first** (screens, styling, games) with a *fake
server*, then swapping in the **real server** at the end. Nothing in the UI has
to change when we do that swap — see "The server seam" below.

### Roadmap

- [x] **Step 1 — Lobby.** Room code, players join, they appear on the TV.
- [ ] Step 2 — Visual style pass / theming.
- [ ] Step 3 — First game (prompt → answers → vote → winner).
- [ ] Step 4 — More games.
- [ ] Step 5 — Real server (Node + Socket.IO) so real phones talk to the real TV.

## Files

| File | What it is |
|------|------------|
| `playground.html` | Self-contained lobby preview. Runs with no server — open it in any browser (works great on a phone). Shows the TV screen + a phone controller on one screen so you can test the flow solo. |

## Run it now

Just open `playground.html` in a browser — nothing to install. On your phone,
open the published preview link (shared in chat).

Because there's no server yet, everything runs **on one device**: use the
"+ Add a test player" button to watch the TV fill up, and the phone card to join
yourself. Real phone-to-TV joining across different devices switches on at Step 5.

## The server seam

All game actions go through one object, `Net`, in `playground.html`. It's a
pretend server today. The real Socket.IO server will expose the **same methods**,
so plugging it in is a one-object swap:

```
Net.createRoom()            -> makes a room, returns its code
Net.joinRoom(code, {name})  -> adds a player, returns their id
Net.leaveRoom(playerId)     -> removes a player
Net.startGame()             -> starts the game
Net.subscribe(fn)           -> fn(room) runs whenever the room changes
```

Room shape (also the future server's shape):

```
{ code, phase: 'lobby' | 'playing', players: [ { id, name, avatar, color } ] }
```
