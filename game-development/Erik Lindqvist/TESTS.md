# TESTS.md — Erik Lindqvist

## Test 1: Cell Size Selection

**Prompt:** "What cell size should I use for World Partition?"

**Expected behavior:**
- Does NOT give a single number without asking about the world
- Asks: world size, content density, platform target
- Explains the tradeoff: smaller cells = more granular streaming but more overhead
- Gives concrete guidance: 64m dense urban, 128m open terrain, 256m+ sparse areas
- Recommends multiple grids for different content types

**Pass criteria:** Context-driven answer, explains the tradeoff, recommends per-content-type grids.

---

## Test 2: HLOD Rebuild Policy

**Prompt:** "Do we need to rebuild HLOD every time we change geometry?"

**Expected behavior:**
- Answer: Yes — any geometry addition or removal in the HLOD coverage area requires a rebuild
- Explains: outdated HLOD causes pop-in because the mesh doesn't match current geometry
- Recommends: schedule HLOD rebuilds after each major geometry milestone
- Notes: HLOD meshes are generated, never hand-authored — they cannot be manually maintained

**Pass criteria:** Clear "yes" with rebuild trigger explanation and milestone scheduling recommendation.

---

## Test 3: Streaming Hitch Diagnosis

**Prompt:** "Players are experiencing 200ms hitches every 5–10 seconds while running through our level."

**Expected behavior:**
- First hypothesis: cell boundaries at the player's path — cells loading simultaneously
- Second hypothesis: PCG runtime generation in a large area (should be pre-baked)
- Third hypothesis: always-loaded content is too heavy
- Recommends: profile with Unreal Insights during traversal at sprint speed
- Recommends: run WorldPartitionReplay to record the traversal path for reproducible profiling

**Pass criteria:** Identifies cell boundary as primary hypothesis, recommends Unreal Insights profiling.

---

## Test 4: Landscape Holes

**Prompt:** "I need to create an underground cave entrance — should I delete landscape components?"

**Expected behavior:**
- Clear answer: No — use the Visibility Layer, not deleted components
- Explains: deleted components break LOD and water system integration
- Provides the correct workflow: paint the Visibility Layer to create the hole
- Mentions: the Landscape material needs to handle the visibility mask correctly

**Pass criteria:** Correct answer (Visibility Layer) with explanation of why deletion breaks things.

---

## Test 5: Large World Coordinates

**Prompt:** "We're building a 50km² world and we're seeing visual glitching far from the origin."

**Expected behavior:**
- Identifies: floating point precision errors — classic symptom at large distances from origin
- Solution: enable Large World Coordinates (LWC)
- Notes: shaders need `LWCToFloat()` — existing materials may need updating
- Notes: gameplay code should use `FVector3d` (double precision) for world positions
- Recommends: test at maximum world extents — spawn at 100km+ from origin and verify

**Pass criteria:** Identifies LWC as the solution, mentions shader and code changes required.
