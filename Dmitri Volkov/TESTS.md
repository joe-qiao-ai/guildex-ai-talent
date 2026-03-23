# TESTS.md — Dmitri Volkov

## Test 1: DataStore Safety

**Prompt:** "Here's my save function: `dataStore:SetAsync(key, data)` — what's wrong with it?"

**Expected behavior:**
- Missing `pcall` — DataStore calls fail; unprotected failures corrupt player data or crash the script
- Missing retry logic — a single transient failure causes data loss
- Correct version: `pcall(dataStore.SetAsync, dataStore, key, data)` with retry
- Also asks: is this called in `BindToClose` as well as `PlayerRemoving`?

**Pass criteria:** Identifies `pcall` as mandatory, mentions retry logic and `BindToClose`.

---

## Test 2: RemoteEvent Exploit

**Prompt:** "A player found a way to give themselves unlimited coins via the shop RemoteEvent. How?"

**Expected behavior:**
- Most likely cause: the server applies a coin amount sent by the client without validating it
- Fix: never trust client-provided amounts — hardcode prices server-side by item ID lookup
- Secondary check: is there a rate limiter? Can the client call the event 1,000 times per second?
- Recommends: audit every `OnServerEvent` handler that touches currency

**Pass criteria:** Identifies client-provided amount as the exploit, recommends server-side price lookup.

---

## Test 3: Script Placement

**Prompt:** "Where should I put a script that handles player combat logic?"

**Expected behavior:**
- Server-side — `ServerStorage/Modules/CombatSystem.lua` as a ModuleScript
- Required by the server bootstrap Script, not accessible to clients
- Explains: combat logic must run on the server — it modifies authoritative state (health)
- Client gets visual feedback via `RemoteEvent:FireClient()` after the server processes the hit

**Pass criteria:** Correct placement in ServerStorage, explains why (authoritative state).

---

## Test 4: BindToClose Importance

**Prompt:** "I save player data in PlayerRemoving. Is that enough?"

**Expected behavior:**
- No — `PlayerRemoving` fires when players leave normally, but NOT reliably on server shutdown
- `game:BindToClose()` must save all remaining players during the shutdown window
- Without `BindToClose`: data from the last X seconds before server restart is lost
- Provides the correct pattern with both hooks

**Pass criteria:** Identifies the shutdown gap, explains `BindToClose` as required.

---

## Test 5: RemoteFunction from Server

**Prompt:** "I want to call `RemoteFunction:InvokeClient()` from the server to ask the client a question."

**Expected behavior:**
- Strong warning: `InvokeClient()` from the server is dangerous
- A malicious or disconnected client can yield the server thread indefinitely
- Alternative 1: use `RemoteEvent:FireClient()` and receive the answer via `RemoteEvent:FireServer()` with a timeout
- Alternative 2: store the answer client-side and have the server query it via a different mechanism
- Does NOT say "it's fine, just be careful"

**Pass criteria:** Clear warning about indefinite yield, provides concrete safer alternative.
