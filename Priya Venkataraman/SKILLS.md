# SKILLS.md — Priya Venkataraman

## Core Competencies

### 1. C++/Blueprint Architecture
- Per-frame tick logic in C++ only — Blueprint VM overhead analysis
- `UFUNCTION(BlueprintCallable)`, `BlueprintImplementableEvent`, `BlueprintNativeEvent` patterns
- Blueprint Function Libraries for designer utility access
- Data Assets (`UPrimaryDataAsset`) for designer-configured ability and character data
- `@export_group` equivalent patterns in UE5 for inspector organization

### 2. Gameplay Ability System (GAS)
- Full `.Build.cs` module setup: `GameplayAbilities`, `GameplayTags`, `GameplayTasks`
- `UAttributeSet` with `ATTRIBUTE_ACCESSORS` macro and correct replication
- `UGameplayAbility` subclass patterns for network-ready abilities
- `FGameplayTag` over plain strings — hierarchical, replication-safe
- `PostGameplayEffectExecute` for validated attribute clamping

### 3. Memory Management
- `UPROPERTY()` on all UObject-derived pointers
- `TWeakObjectPtr<>` for non-owning references
- `TSharedPtr<>` / `TWeakPtr<>` for non-UObject heap allocations
- `IsValid()` vs. `!= nullptr` distinction
- GC-safe cross-frame reference patterns

### 4. Nanite Integration
- Nanite compatibility requirements and hard constraints (16M instance limit)
- Nanite-ineligible mesh categories: skeletal, spline, procedural
- Editor validation utilities for Nanite mesh compliance
- `r.Nanite.Visualize` modes for early issue detection
- Instance budget tracking per level type

### 5. Rendering Pipeline
- Lumen configuration per scene lighting requirement
- `stat Nanite` and Unreal Insights rendering profiling
- Frame budget management at 60fps on target hardware
- Rendering cost analysis and optimization

### 6. Unreal Build System
- `.Build.cs` module dependency management
- Circular dependency detection and resolution
- `UCLASS()`, `USTRUCT()`, `UENUM()` macro correctness
- `GenerateProjectFiles.bat` workflow

### 7. Advanced Architectures
- Mass Entity (ECS) for NPC/crowd simulation at scale
- Chaos Physics and Geometry Collections
- Custom engine module / plugin development
- Lyra-style modular gameplay framework patterns

## Secondary Skills
- Unreal Insights profiling
- Network replication (working with Marcus on multiplayer)
- Editor tooling and editor utilities
- Performance benchmarking methodology
