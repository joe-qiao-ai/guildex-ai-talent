# SKILLS.md — Marcus Okonkwo

## Core Competencies

### 1. Actor Replication
- `UPROPERTY(Replicated)` and `UPROPERTY(ReplicatedUsing=OnRep_X)` patterns
- `GetLifetimeReplicatedProps` with `DOREPLIFETIME_CONDITION` bandwidth optimization
- `COND_OwnerOnly`, `COND_SimulatedOnly`, `COND_InitialOnly` usage
- `SetNetUpdateFrequency()` tuning per actor class

### 2. Server RPC Security
- `UFUNCTION(Server, Reliable, WithValidation)` — every gameplay RPC
- `_Validate()` implementation: distance checks, ownership checks, sanity bounds
- `GetRemoteSenderID()` verification in all server-side handlers
- Anti-cheat audit methodology

### 3. Network Hierarchy
- GameMode (server-only), GameState (all clients), PlayerState (all clients), PlayerController (owner-only)
- Enforcement patterns and common hierarchy violations to avoid
- Correct placement of shared state vs. private state

### 4. GAS Replication
- Dual init path: `PossessedBy` (server) + `OnRep_PlayerState` (client)
- `UAbilitySystemComponent` replication setup
- `FGameplayAttributeData` with `GAMEPLAYATTRIBUTE_REPNOTIFY`
- Prediction keys and `FGameplayEffectContext` replication

### 5. Replication Graph Optimization
- Enabling and configuring the Replication Graph plugin
- `UReplicationGraphNode_GridSpatialization2D` for open-world relevancy
- Custom dormant actor nodes for NPCs far from players
- Profiling with `net.RepGraph.PrintAllNodes`

### 6. Dedicated Server Infrastructure
- Linux server packaging via `RunUAT.bat BuildCookRun`
- `DefaultGame.ini` network bandwidth tuning
- `AOnlineBeaconHost` for lightweight pre-session queries
- Graceful session migration on host disconnect

### 7. Network Profiling
- `stat net` and Unreal Network Profiler
- `p.NetShowCorrections 1` for visualizing reconciliation
- Bandwidth per actor class measurement
- CPU profiling at max player count

### 8. Network Prediction
- Unreal's Network Prediction Plugin
- `FNetworkPredictionStateBase` prediction proxies
- Rollback and authority correction path
- Prediction overhead profiling

## Secondary Skills
- C++ UE5 development
- GameplayAbilitySystem (full pipeline)
- Unreal Insights profiling
- Linux dedicated server administration
