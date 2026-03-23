# TESTS.md — Marcus Okonkwo

## Evaluation Criteria

Each test evaluates Marcus's core capabilities. Grade: Pass / Partial / Fail.

---

## Test 1: RPC Security Audit

**Prompt:** "Here's a Server RPC for picking up items. Is it secure?"
```cpp
UFUNCTION(Server, Reliable)
void ServerPickupItem(int32 ItemID);
void ServerPickupItem_Implementation(int32 ItemID)
{
    GiveItemToPlayer(ItemID);
}
```

**Expected behavior:**
- Immediately flags: missing `WithValidation`, no `_Validate()` function
- Points out: any client can request any ItemID including items they haven't seen
- Points out: no distance check — client could pick up items on the other side of the map
- Provides corrected version with `WithValidation` and range/ownership validation
- Mentions this is a cheat vector for inventory duplication

**Pass criteria:** Identifies all three issues, provides corrected code.

---

## Test 2: Network Hierarchy Placement

**Prompt:** "Where should I store the team score — in GameMode, GameState, PlayerState, or the player's Character?"

**Expected behavior:**
- Answer: `GameState` — it's shared world state that all clients need to see
- Explains: GameMode is server-only and never replicated
- Explains: PlayerState is per-player data, not team data
- Explains: Character is too volatile and may not be spawned when score is relevant
- Offers code example showing `UPROPERTY(Replicated)` in GameState with `OnRep` for UI updates

**Pass criteria:** Correct answer with clear justification for each alternative.

---

## Test 3: GAS Replication Debugging

**Prompt:** "Attributes work on the server but clients always show the default values. What's wrong?"

**Expected behavior:**
- First hypothesis: missing `OnRep_PlayerState` dual init path
- Second hypothesis: `GAMEPLAYATTRIBUTE_REPNOTIFY` macro missing in AttributeSet
- Third hypothesis: `GetLifetimeReplicatedProps` not calling `DOREPLIFETIME` for attribute data
- Walks through diagnosis steps, provides fix for each scenario
- Mentions: add debug command to dump attribute values on both server and client to confirm

**Pass criteria:** Identifies the dual-init issue as primary suspect, provides complete fix.

---

## Test 4: Bandwidth Optimization

**Prompt:** "I have 500 NPC enemies in an open world all using default replication settings. Players say it lags."

**Expected behavior:**
- Diagnoses: default `NetUpdateFrequency` is too high for non-player actors
- Recommends: 20Hz for nearby NPCs, 5Hz for distant, dormant for inactive
- Recommends: Replication Graph with `UReplicationGraphNode_GridSpatialization2D`
- Recommends: `COND_SkipOwner` for state clients can predict locally
- Quantifies expected improvement
- Does NOT suggest just "disabling replication" without architectural replacement

**Pass criteria:** Provides a concrete optimization plan with Replication Graph guidance.

---

## Test 5: Latency Behavior

**Prompt:** "My game feels fine on LAN but terrible at 150ms. What architecture changes do I need?"

**Expected behavior:**
- Identifies: client prediction is missing for movement
- Recommends: Network Prediction Plugin for physics-driven movement
- Recommends: unreliable RPCs for high-frequency hints, reliable only for state changes
- Recommends: cosmetic-only client-side prediction for non-gameplay effects
- Mentions: simulate 150ms and 200ms in PIE Network Emulation settings before claiming a fix works

**Pass criteria:** Covers prediction, RPC reliability tiers, and testing methodology.
