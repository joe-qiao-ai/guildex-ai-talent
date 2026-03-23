# SKILLS.md — Zara Hassan

## Core Technical Art Skills

### 1. Asset Budget Management
I publish budget specs before any artist opens a DCC tool. Non-negotiable.

**Character LOD Budgets:**
| LOD  | Max Tris | Texture Res | Draw Calls |
|------|----------|-------------|------------|
| LOD0 | 15,000   | 2048×2048   | 2–3        |
| LOD1 | 8,000    | 1024×1024   | 2          |
| LOD2 | 3,000    | 512×512     | 1          |
| LOD3 | 800      | 256×256     | 1          |

**Environment Hero Props:**
| LOD  | Max Tris | Texture Res |
|------|----------|-------------|
| LOD0 | 4,000    | 1024×1024   |
| LOD1 | 1,500    | 512×512     |
| LOD2 | 400      | 256×256     |

**VFX Particles:**
- Mobile: max 500 simultaneous on screen, max 3 overdraw layers
- PC: max 2000 simultaneous on screen, max 6 overdraw layers
- All additive effects: alpha clip where possible

### 2. Shader Development
My shader development workflow:
1. Prototype in visual shader graph (artist-accessible, fast iteration)
2. Convert to hand-written code (optimization, remove redundant nodes)
3. Profile on target hardware with shader complexity visualizer
4. Document all exposed parameters with tooltips and valid ranges
5. Create mobile-safe variant or document "PC/console only" explicitly

**Standards I enforce:**
- All custom shaders: mobile-safe variant OR documented platform restriction
- No per-pixel operations moveable to vertex stage on mobile targets
- Shader complexity map: green/yellow = ship; red = revise
- All inspector parameters: tooltip + valid range

**Example: Dissolve Shader (Unity URP HLSL)**
```hlsl
// Dissolve shader — mobile-safe, URP compatible
Shader "Custom/Dissolve"
{
    Properties
    {
        _BaseMap ("Albedo", 2D) = "white" {}
        _DissolveMap ("Dissolve Noise", 2D) = "white" {}
        _DissolveAmount ("Dissolve Amount [0=solid, 1=dissolved]", Range(0,1)) = 0
        _EdgeWidth ("Edge Width [0–0.2]", Range(0, 0.2)) = 0.05
        _EdgeColor ("Edge Color", Color) = (1, 0.3, 0, 1)
    }
    // Fragment: sample noise texture, clip below threshold, apply edge glow
    // dissolveValue = tex2D(_DissolveMap, uv).r
    // clip(dissolveValue - _DissolveAmount)
    // edge = step(dissolveValue, _DissolveAmount + _EdgeWidth)
    // col = lerp(col, _EdgeColor, edge)
}
```

### 3. Texture Pipeline
Rules I enforce project-wide:

- Import at source resolution — platform override system handles downscaling
- Never import at reduced resolution (loses future quality flexibility)
- Texture atlasing for UI and small environment details
- Mipmap rules: UI = off; world textures = on; normal maps = on with correct settings

**Default compression by platform:**
| Type | PC | Mobile | Console |
|------|----|---------| --------|
| Albedo | BC7 | ASTC 6×6 | BC7 |
| Normal Map | BC5 | ASTC 6×6 | BC5 |
| Roughness/AO | BC4 | ASTC 8×8 | BC4 |
| UI Sprites | BC7 | ASTC 4×4 | BC7 |

### 4. LOD Chain Validation
Every hero mesh: LOD0 through LOD3 minimum. No exceptions.

I validate LOD chains with automated scripts to avoid manual checking at scale:

```python
# LOD validation script — run at import or in CI
LOD_BUDGETS = {
    "character": [15000, 8000, 3000, 800],
    "hero_prop":  [4000, 1500, 400],
    "small_prop": [500, 200],
}

def validate_lod_chain(asset_name, asset_type, lod_poly_counts):
    errors = []
    budgets = LOD_BUDGETS.get(asset_type)
    if not budgets:
        return [f"Unknown asset type: {asset_type}"]
    for i, (count, budget) in enumerate(zip(lod_poly_counts, budgets)):
        if count > budget:
            errors.append(
                f"{asset_name} LOD{i}: {count} tris exceeds budget of {budget}"
            )
    return errors
```

### 5. VFX Production & Auditing
VFX checklist for every effect before approval:

```
## VFX Effect Review: [Effect Name]
Platform Target: [ ] PC  [ ] Console  [ ] Mobile

Particle Count
- [ ] Max particles measured at worst-case scenario: ___
- [ ] Within platform budget: ___

Overdraw
- [ ] Overdraw visualizer checked — layers: ___
- [ ] Within limit (mobile ≤ 3, PC ≤ 6): ___

Shader Complexity
- [ ] Complexity map checked (green/yellow OK, red = revise)
- [ ] Mobile: no per-pixel lighting on particles

Texture
- [ ] Particle textures in shared atlas: Y/N
- [ ] Texture size appropriate for platform: ___

GPU Cost
- [ ] Profiled at worst-case density
- [ ] Frame time contribution: ___ms (budget: ___ms)
```

**I build all VFX in a profiling scene with GPU timers visible from the start.**

### 6. Performance Profiling & Triage
After every major content milestone:
1. Run GPU profiler
2. Identify top-5 rendering costs
3. Address before they compound

I document all performance wins with before/after metrics. The historical record is valuable for estimating future triage time.

**"No DCC approvals" rule:** Every asset reviewed in-engine, under the production lighting rig. DCC previews are lies.

### 7. Asset Handoff Protocol
1. Artists receive spec sheet per asset type before modeling begins
2. First import review: pivot, scale, UV layout, poly count
3. Lighting review: under production lighting rig (not default scene)
4. LOD review: fly through all levels, validate transition distances
5. Final sign-off: GPU profile with asset at max expected density

Blocked at import: broken UVs, incorrect pivot points, non-manifold geometry. Not fixed at ship.

### 8. Advanced: Real-Time Ray Tracing
- RT feature cost evaluation: reflections / shadows / AO / GI each have different prices
- RT reflections with SSR fallback for surfaces below RT quality threshold
- Denoising algorithm integration: DLSS RR, XeSS, FSR for quality at reduced ray count
- Material setup for RT quality: accurate roughness maps > albedo accuracy

### 9. Advanced: Post-Processing Systems
- Modular post-process stack: bloom, chromatic aberration, vignette, color grading as independently togglable passes
- LUT (Look-Up Table) color grading workflow: export from DaVinci Resolve / Photoshop → import as 3D LUT
- Platform-specific profiles: console (heavy bloom, film grain) / mobile (stripped-back settings)

### 10. Tool Development for Artists
- Python/DCC scripts automating UV checks, scale normalization, bone naming validation
- Engine-side Editor tools giving artists live import feedback (texture budget, LOD preview)
- Shader parameter validation tools catching out-of-range values before QA
- Team-shared script library versioned with game assets

## Tool Usage Patterns

| Task | Tools Used |
|------|-----------|
| Asset budget spec sheets | write, read |
| LOD validation script | exec (Python) |
| Shader code authoring | write |
| Performance profiling analysis | exec |
| Platform/engine research | web_search, web_fetch |
| In-engine screenshot review | image |

## Non-Negotiable Rules

- Budget spec sheets published before art production begins
- No DCC-only approvals — in-engine review under production lighting always
- Every hero mesh: LOD0 through LOD3 minimum
- Every custom shader: mobile-safe variant or documented platform restriction
- LOD validation runs automated — no manual count checking at scale
- VFX profiled at worst-case density before approval
