# TESTS.md — Amara Diallo

## Test 1: RPC Mode Selection

**Prompt:** "I need to tell all clients to play a hit effect. Which RPC mode should I use?"

**Expected behavior:**
- `@rpc("authority")` — only the authority (server) can call it, all clients receive it
- `"unreliable"` for visual effects — packet loss for a hit spark is acceptable, saves bandwidth
- Explains the alternative: `"call_local"` if the server should also see the effect locally
- Warns against `"any_peer"` for this use case — clients shouldn't be able to trigger hit effects on each other

**Pass criteria:** Correct mode (`authority`, `unreliable`), explains why.

---

## Test 2: Authority Assignment

**Prompt:** "How do I make sure each player controls only their own character?"

**Expected behavior:**
- Name each player node with the peer ID as a string: `player.name = str(peer_id)`
- Call `set_multiplayer_authority(peer_id)` immediately after `add_child()`
- Guard all state mutations with `is_multiplayer_authority()`
- Only the matching peer can execute authority-guarded code
- Explains: this means `_physics_process` on a player node runs authoritatively on that peer's machine

**Pass criteria:** Correct authority assignment pattern with `is_multiplayer_authority()` guard.

---

## Test 3: Missing Validation

**Prompt:** "Here's my server RPC: `func take_damage(amount: float)` — is it secure?"

**Expected behavior:**
- No — missing sender validation
- A client can call `take_damage()` on any player node with any value
- Fix: check `multiplayer.is_server()`, get `multiplayer.get_remote_sender_id()`, verify caller owns this player
- Fix: validate `amount` is within a reasonable range (no negative damage or massive one-shot values)

**Pass criteria:** Identifies both missing checks, provides fix.

---

## Test 4: Spawner vs Manual add_child

**Prompt:** "My player nodes appear on the server but not on clients."

**Expected behavior:**
- Diagnosis: `add_child()` used without `MultiplayerSpawner` — local only
- Fix: ensure `MultiplayerSpawner` is in the scene with the player scene registered
- When server calls `add_child()` with Spawner present, all peers receive the node
- Checks: is the player scene registered in the Spawner's `spawn_path` list?

**Pass criteria:** Identifies Spawner as the missing piece, explains the fix.

---

## Test 5: Latency Testing

**Prompt:** "The multiplayer works fine in my tests. Can I ship it?"

**Expected behavior:**
- Asks: what latency did you test at?
- If localhost only: not sufficient — requires 150ms minimum, ideally 200ms simulated latency
- Explains: many multiplayer bugs only appear at realistic latency: RPC ordering, authority mismatches, prediction errors
- Recommends: use Godot's built-in Network Emulation settings to add artificial latency in testing
- Recommends: test reconnection — what happens when a client drops and rejoins?

**Pass criteria:** Asks about latency, recommends 150–200ms test, mentions reconnection.
