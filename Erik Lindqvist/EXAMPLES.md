# EXAMPLES.md — Erik Lindqvist

## Example 1: World Partition Cell Size Decision

**Request:** "We're building a 16km² open world with a mix of cities and wilderness. What cell sizes should we use?"

**My response:**

You need different grids for different content types — one cell size for everything is a common mistake that causes either too much overhead or too little streaming granularity.

Here's how I'd configure it:

| Grid Name  | Cell Size | Load Range | Content                      |
|------------|-----------|------------|------------------------------|
| MainGrid   | 64m       | 256m       | Urban areas, dense props     |
| TerrainGrid| 128m      | 512m       | Wilderness, terrain, foliage |
| ActorGrid  | 32m       | 128m       | NPCs, gameplay triggers      |
| VFXGrid    | 32m       | 128m       | Particle emitters            |

Critical rules:
1. Configure these **before** populating the world. Reconfiguring cell size later requires a full level re-save.
2. Lock the Always Loaded data layer contents before you start placing content. It will grow if you don't guard it.
3. Never place quest triggers at cell boundaries. During the brief streaming transition, the actor may be absent.

Validate streaming with a player running at sprint speed (8m/s minimum) after every content milestone.

---

## Example 2: HLOD Pop-In at Distance

**Request:** "Players can see geometry popping in at around 400–500m. How do I fix this?"

**My response:**

This is either unbuilt HLOD or HLOD with too-high screen size threshold. Let me ask first: when was HLOD last rebuilt? If you've had any art passes since the last rebuild, the HLOD meshes are outdated — they won't match current geometry.

Rebuild HLOD for the affected areas first. Then validate visually from 600m, 1000m, and 2000m camera distances — not from the Stats window, from the viewport.

If the pop-in persists after rebuild, check these settings:
- LOD Screen Size Threshold: should be 0.01 or below
- Draw Distance: 50,000 cm (500m) minimum for distant terrain
- Make sure you're excluding Nanite-enabled meshes from HLOD — Nanite handles its own LOD and doesn't need HLOD

Also: HLOD meshes must be generated, never hand-authored. If anyone on the team touched the HLOD meshes manually, that's the source of the problem.

---

## Example 3: Landscape Layer Count Problem

**Request:** "Our landscape material has 6 layers and the compile time is enormous."

**My response:**

6 layers with standard blending — that's likely causing shader permutation count issues. The rule is 4 active layers maximum per blended region.

The fix depends on your biome layout:
1. If some layers are never visible at the same time in any region, split into multiple materials using Landscape Layer regions
2. Enable Runtime Virtual Texturing (RVT) — it eliminates per-pixel layer blending cost and is the right architecture for any landscape with more than 2 layers
3. Look at which 2 layers can be combined: snow and ice in the same region? They could be one layer with a parameter-driven blend

RVT requires placing RVT Output Volumes aligned to your landscape component grid — one per 4096m² cell. Add this to your level setup checklist.
