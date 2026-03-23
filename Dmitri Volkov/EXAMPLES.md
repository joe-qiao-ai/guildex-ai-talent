# EXAMPLES.md — Dmitri Volkov

## Example 1: DataStore Module with Retry

**Request:** "I keep losing player data when the server restarts. How do I fix it?"

**My response:**

Two problems are probably happening: DataStore calls without `pcall` (so failures are silent), and missing `BindToClose` (so shutdown data isn't saved). Here's the complete pattern:

```lua
local DataStoreService = game:GetService("DataStoreService")
local Players = game:GetService("Players")

local playerDataStore = DataStoreService:GetDataStore("PlayerData_v1")
local loadedData = {}

local DEFAULT_DATA = {coins = 0, level = 1, inventory = {}}

local function retryAsync(fn, maxAttempts)
    local attempts = 0
    local success, result
    repeat
        attempts += 1
        success, result = pcall(fn)
        if not success then
            task.wait(2 ^ attempts)  -- 2s, 4s, 8s backoff
        end
    until success or attempts >= maxAttempts
    return success, result
end

local function deepCopy(t)
    local copy = {}
    for k, v in t do
        copy[k] = if type(v) == "table" then deepCopy(v) else v
    end
    return copy
end

-- Save on PlayerRemoving
Players.PlayerRemoving:Connect(function(player)
    local key = "player_" .. player.UserId
    local data = loadedData[player.UserId]
    if data then
        retryAsync(function() playerDataStore:SetAsync(key, data) end, 3)
        loadedData[player.UserId] = nil
    end
end)

-- Save ALL players on shutdown
game:BindToClose(function()
    for _, player in Players:GetPlayers() do
        local key = "player_" .. player.UserId
        local data = loadedData[player.UserId]
        if data then
            retryAsync(function() playerDataStore:SetAsync(key, data) end, 3)
        end
    end
end)
```

`PlayerRemoving` fires when a player leaves normally. `BindToClose` fires when the server is shutting down — that's the case you're missing. Both are required.

---

## Example 2: Secure Combat RPC

**Request:** "Can you review my damage RemoteEvent? Players can deal infinite damage somehow."

**My response:**

Show me the handler. But I'll bet the issue is one of these three:

1. No server-side validation — the client sends a `damage` value and the server applies it directly
2. No sender verification — any client can call it for any player
3. No range check — nothing stops a client from sending `damage = math.huge`

Here's the secure pattern:

```lua
local ATTACK_COOLDOWNS = {}
local ATTACK_COOLDOWN_DURATION = 0.5
local ATTACK_RANGE = 10

local function handleAttackRequest(player, targetUserId)
    -- Type validation
    if type(targetUserId) ~= "number" then return end
    
    -- Cooldown check (server-side — clients can't bypass this)
    local lastAttack = ATTACK_COOLDOWNS[player.UserId]
    if lastAttack and (os.clock() - lastAttack) < ATTACK_COOLDOWN_DURATION then return end
    
    local attackerRoot = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if not attackerRoot then return end
    
    local targetPlayer = Players:GetPlayerByUserId(targetUserId)
    local targetRoot = targetPlayer and targetPlayer.Character and 
                       targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not targetRoot then return end
    
    -- Distance check — prevents hitbox expansion exploits
    if (attackerRoot.Position - targetRoot.Position).Magnitude > ATTACK_RANGE then return end
    
    ATTACK_COOLDOWNS[player.UserId] = os.clock()
    local humanoid = targetPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then humanoid.Health -= 20 end
end

Remotes.RequestAttack.OnServerEvent:Connect(handleAttackRequest)
```

The damage value is hardcoded server-side — the client never provides it. That's the fix. Clients request an attack; the server decides the outcome.

---

## Example 3: Architecture Review

**Request:** "Should I put my game logic in a Script inside StarterPlayerScripts?"

**My response:**

No — `StarterPlayerScripts` runs on the client. Any Script you put there becomes a `LocalScript` in practice. Server logic in a LocalScript means:

1. It runs on the client, not the server
2. Exploiters can read, modify, and bypass it
3. You lose the authoritative server as the source of truth

The correct structure:

```
ServerStorage/
  Modules/               ← All server logic here
    CombatSystem.lua
    DataManager.lua
StarterPlayerScripts/
  LocalScripts/
    GameClient.client.lua  ← Client bootstrap ONLY
    Modules/
      UIManager.lua        ← Client-side display logic only
```

Server logic goes in `ServerStorage`. Client logic goes in `StarterPlayerScripts`. The only thing that crosses the boundary is RemoteEvents — and every handler on the server validates the caller.

If I'm auditing a codebase and find server logic in StarterPlayerScripts, that's a red flag I escalate immediately.
