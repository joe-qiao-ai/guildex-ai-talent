# TESTS.md — Isabela Reyes

## Test Suite: Unity Shader Graph Artist

These tests verify Isabela correctly applies shader architecture, pipeline rules, and performance budgeting.

---

### Test 1: Sub-Graph Requirement

**Input:**
> I have dissolve logic copied into 5 different shaders. Should I refactor?

**Expected behaviors:**
- [ ] Immediately recommends creating a `DissolveCore` Sub-Graph
- [ ] Explains the maintenance problem: 5 copies = 5 places to fix bugs
- [ ] Describes what inputs/outputs the Sub-Graph should expose
- [ ] Notes that visual variation stays in per-material parameters (Dissolve Map texture), not in the logic

**Red flags:**
- Suggests keeping separate implementations "for flexibility"
- Doesn't mention Sub-Graphs at all

---

### Test 2: URP vs. HDRP API

**Input:**
> I'm trying to use `OnRenderImage` for a screen-space effect in my URP project.

**Expected behaviors:**
- [ ] Clearly states `OnRenderImage` is built-in pipeline only — not available in URP
- [ ] Provides the correct URP alternative: `ScriptableRendererFeature` + `ScriptableRenderPass`
- [ ] Notes that HDRP uses a completely different API (`CustomPassVolume` + `CustomPass`)
- [ ] Provides at least a code sketch of the URP renderer feature structure

**Red flags:**
- Says "just add it to the camera component"
- Doesn't distinguish URP and HDRP APIs

---

### Test 3: Mobile Budget Assessment

**Input:**
> My character shader has 12 texture samples in the fragment stage. Is that okay for mobile?

**Expected behaviors:**
- [ ] States the mobile opaque fragment texture sample limit (8-32 depending on tier, typically 8 for safe budget)
- [ ] Flags 12 samples as over the conservative mobile budget
- [ ] Suggests specific reduction strategies: texture packing (ORM channels), baked detail textures, mobile material variants
- [ ] Does NOT just say "it depends" — gives concrete guidance

**Red flags:**
- Says 12 samples is fine without any qualification
- Doesn't mention texture packing as a reduction strategy

---

### Test 4: Alpha Blend vs. Alpha Clip

**Input:**
> For our foliage, should I use Alpha Blend or Alpha Clip transparency?

**Expected behaviors:**
- [ ] Recommends Alpha Clip (Alpha Clipping) where visual quality allows
- [ ] Explains that Alpha Blend introduces depth sorting problems and overdraw costs
- [ ] Notes Alpha Clip works with depth writes (no sorting needed)
- [ ] May mention Alpha-to-Coverage as a dithered middle ground
- [ ] Asks about visual quality requirement before making a final recommendation

**Red flags:**
- Recommends Alpha Blend for foliage without discussing overdraw
- Doesn't explain the depth sorting issue with Alpha Blend

---

### Test 5: HLSL Macro Compatibility

**Input:**
> I wrote a URP HLSL shader using `sampler2D` and `tex2D()`. It compiles but materials look black on some platforms.

**Expected behaviors:**
- [ ] Identifies `sampler2D` / `tex2D` as built-in pipeline macros not compatible with SRP
- [ ] Provides the correct SRP replacements: `TEXTURE2D` / `SAMPLER` / `SAMPLE_TEXTURE2D`
- [ ] Notes the `CBUFFER_START(UnityPerMaterial)` requirement for material properties
- [ ] Explains the `Core.hlsl` include requirement for SRP macros

**Red flags:**
- Says `sampler2D` is fine in URP
- Doesn't mention the SRP macro system

---

### Test 6: Blackboard Parameter Documentation

**Input:**
> I finished my Shader Graph and exposed about 15 parameters. Anything else before I hand it off to artists?

**Expected behaviors:**
- [ ] Asks whether all parameters have Blackboard tooltips set
- [ ] States that undocumented parameters are a blocker for handoff
- [ ] Recommends reviewing which parameters artists actually need vs. internal values that should be hidden in Sub-Graphs
- [ ] Suggests creating a Material Instance setup guide for the most common use case

**Red flags:**
- Says the shader is ready without checking documentation
- Doesn't mention Blackboard tooltips

---

### Test 7: Performance Before Visual

**Input:**
> I want to add a procedural animated water surface with real-time wave normals, subsurface scattering approximation, caustic projection, and screen-space reflections.

**Expected behaviors:**
- [ ] Does NOT immediately say "here's how to build all of that"
- [ ] Asks about target platform and performance budget first
- [ ] Breaks down the cost of each feature (wave normals = compute/texture, SSR = expensive screen-space pass)
- [ ] Prioritizes the features by visual impact vs. cost
- [ ] Recommends building the most impactful features first within budget

**Red flags:**
- Implements all features without budget discussion
- Doesn't ask about platform target

---

### Test 8: OpenClaw Tool Usage

**Input:**
> Can you review my shader file at `Assets/Shaders/CharacterLit.hlsl`?

**Expected behaviors:**
- [ ] Uses `read` tool to load the file
- [ ] Reviews for: SRP macro usage, `cbuffer` property matching, texture sample count, mobile derivative usage
- [ ] Uses `edit` tool for inline corrections
- [ ] Does NOT ask user to paste the file if a path is available

**Red flags:**
- Asks user to paste the shader code
- Reviews generically without reading the actual file
