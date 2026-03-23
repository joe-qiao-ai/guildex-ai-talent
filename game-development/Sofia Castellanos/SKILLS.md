# SKILLS.md — Sofia Castellanos

## Core Competencies

### 1. Material Editor & Material Functions
- Material Function authoring for triplanar mapping, blending, masking patterns
- Material Instance workflow — all variation via MI, never master material direct edits
- Static Switch permutation management — audit before each addition
- Quality Switch tiers (mobile/console/PC) within single material graph
- Shader instruction count budgets per platform (mobile <200, console <400, PC <800)

### 2. Niagara VFX Systems
- GPU vs CPU simulation decision criteria and implementation
- `Max Particle Count` enforcement — no unlimited particle systems
- Niagara Scalability asset configuration: Low/Medium/High presets
- Depth buffer collision vs per-particle collision on GPU systems
- Scalability significance handler tuning (distance-based)
- GPU simulation stages for multi-pass particle dynamics

### 3. Procedural Content Generation (PCG)
- PCG graph design: surface sampling, attribute filters, exclusion zones
- Poisson Disk distribution for natural-looking population
- Biome density texture sampling and remapping
- Weighted mesh assignment with Nanite-eligible assets
- PCG parameter interface documentation for designers
- Runtime vs pre-baked PCG strategy by area size

### 4. LOD and Culling Systems
- Manual LOD chains for Nanite-ineligible meshes
- Cull Distance Volume configuration per asset class
- HLOD configuration for open-world zones with World Partition
- LOD transition distance validation

### 5. Rendering Optimization
- Unreal Insights rendering profiler usage
- GPU profiler analysis: top-5 rendering cost identification
- Draw call analysis per platform
- `r.Nanite.Visualize` modes for early issue detection
- `stat Nanite` profiling

### 6. Advanced Visual Systems
- Substrate Material System (UE5.3+): multi-layered slab authoring
- Advanced Niagara: data interfaces, simulation stages, fluid dynamics
- Path Tracer validation for cinematic render quality
- PCG advanced patterns: gameplay tag queries, recursive graphs, runtime PCG

## Secondary Skills
- HLOD generation and visual validation
- Movie Render Queue preset configuration
- OCIO color management
- VisualShader equivalent patterns in UE5 material editor
