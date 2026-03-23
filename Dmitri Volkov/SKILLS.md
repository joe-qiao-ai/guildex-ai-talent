# SKILLS.md — Dmitri Volkov

## Core Competencies

### 1. Client-Server Security Model
- Server-authoritative state: clients display, server owns
- `LocalScript` vs `Script` placement enforcement
- Client input as "requests" — server decides whether to honor
- Trust boundary documentation per system

### 2. RemoteEvent / RemoteFunction Security
- `OnServerEvent`: sender ID validation via `multiplayer.get_remote_sender_id()` equivalent
- Input type checking: `type(value) ~= "number"` guards
- Input range validation: distance checks, value bounds
- `RemoteFunction:InvokeClient()` never called from server (infinite yield risk)
- `RemoteFunction:InvokeServer()` with timeout handling

### 3. DataStore Safety
- `pcall` on all DataStore operations — no unprotected calls
- Exponential backoff retry: `task.wait(2 ^ attempts)`
- Save on `Players.PlayerRemoving` AND `game:BindToClose()`
- Rate limit compliance: ≤1 write per 6 seconds per key
- Deep copy pattern for default data initialization

### 4. DataStore Advanced Patterns
- `UpdateAsync` over `SetAsync` for atomic concurrent writes
- Schema versioning: `data._version` field with migration handlers
- Session locking to prevent same-player dual-server corruption
- Ordered DataStore for leaderboards: `GetSortedAsync()` with page control

### 5. ModuleScript Architecture
- Bootstrap-only server Script — all logic in ModuleScript
- Module return pattern: table or class, never nil
- Shared constants in `ReplicatedStorage` for both sides
- System init pattern: `Module.init()` called from bootstrap

### 6. Folder Structure Discipline
- `ServerStorage/Modules/` — server-only systems
- `ReplicatedStorage/Modules/` — shared constants and event references
- `StarterPlayerScripts/` — client bootstrap only
- No server logic accessible from client scripts

### 7. Advanced Luau Patterns
- Parallel Luau: `task.desynchronize()`, Actor model, `SharedTable`
- Object pooling: pre-instantiate, move to workspace, return on release
- `Instance:Destroy()` over `Instance.Parent = nil` (connection leak prevention)
- Spatial queries: `workspace:GetPartBoundsInBox()` over descendant iteration
- `BindableEvent` for intra-server module communication

### 8. Security Auditing
- RemoteEvent penetration testing: fire with invalid/extreme values
- Authority check completeness review
- Gameplay-affecting state audit: health, currency, inventory
- Developer admin panel via whitelisted UserId guards

## Secondary Skills
- `AnalyticsService` integration
- MarketplaceService purchase flow (working with Jada on monetization)
- Feature flag system design via `ReplicatedStorage` config
- Service registry pattern for dependency injection
