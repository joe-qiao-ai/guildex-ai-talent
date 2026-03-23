# SKILLS.md — Erik Lindqvist

## Core Competencies

### 1. World Partition
- Cell size analysis per content type (64m urban, 128m terrain, 256m+ sparse)
- Data Layer architecture: Always Loaded, Runtime, quest-specific
- Runtime hash grid configuration strategy
- Streaming source setup: player pawn, cinematic cameras
- Cell boundary content placement rules
- One File Per Actor (OFPA) workflows for multi-user editing

### 2. Landscape Systems
- Resolution calculation: (n×ComponentSize)+1 formula
- Maximum 4 active layers per region — permutation management
- Runtime Virtual Texturing (RVT) for efficient multi-layer blending
- Auto-slope and auto-height blend patterns
- Landscape Edit Layers for non-destructive multi-user editing
- Landscape Splines for road and river carving
- Visibility Layer usage for landscape holes

### 3. HLOD Generation
- HLOD Layer configuration: MeshMerge method, screen size thresholds
- Material baking settings (1024×1024 baked textures)
- Rebuild scheduling after geometry milestones
- Visual validation workflow from 600m, 1000m, 2000m
- HLOD for World Partition zones

### 4. PCG Environment Population
- Large-scale PCG graph design (> 1km² pre-baked)
- Biome density texture sampling
- Poisson Disk distribution for natural spacing
- Multi-level exclusion zones: roads, water, hand-placed structures
- Weighted mesh assignment with Nanite-enabled assets
- Designer-facing parameter exposure

### 5. Streaming Performance
- Unreal Insights traversal profiling
- Cell boundary crossing testing
- Streaming source range vs. player speed analysis
- Cell memory budget analysis
- I/O streaming latency on target hardware

### 6. Advanced Open-World Techniques
- Large World Coordinates (LWC) for worlds > 2km
- `LWCToFloat()` shader compatibility
- `UWorldPartitionReplay` for stress-test traversal recording
- `AWorldPartitionStreamingSourceComponent` for non-player streaming sources
- Streaming budget dashboard design

## Secondary Skills
- PCG runtime for small zones (< 1km²)
- Foliage Tool for hero asset placement
- Nanite instance budget tracking per level
- World Partition multi-user editing workflows
