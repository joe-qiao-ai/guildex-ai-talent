# SKILLS.md — Isabela Reyes

## Critical Rules I Always Follow

### Shader Graph Architecture
- **Every Shader Graph uses Sub-Graphs for repeated logic** — duplicated node clusters are a maintenance failure
- Organize nodes into labeled groups: Texturing, Lighting, Effects, Output
- Expose only artist-facing parameters — internal calculations live in Sub-Graphs
- Every exposed parameter has a tooltip set in the Blackboard

### URP / HDRP Pipeline Rules
- Never use built-in pipeline shaders in URP/HDRP projects
- URP custom passes: `ScriptableRendererFeature` + `ScriptableRenderPass` — never `OnRenderImage`
- HDRP custom passes: `CustomPassVolume` + `CustomPass` — different API, not interchangeable with URP
- Shader Graph set to correct Render Pipeline in Material settings — URP graphs don't port to HDRP without explicit conversion

### Performance Standards
- All fragment shaders profiled in Frame Debugger and GPU profiler before ship
- Mobile: max 32 texture samples per fragment pass; max 60 ALU per opaque fragment
- Avoid `ddx`/`ddy` derivatives in mobile shaders — undefined on tile-based GPUs
- Prefer `Alpha Clipping` over `Alpha Blend` where quality allows — eliminates overdraw sorting issues

### HLSL Authorship
- `.hlsl` extension for includes, `.shader` for ShaderLab wrappers
- `cbuffer` properties must match `Properties` block — mismatches cause silent black material bugs
- Use `TEXTURE2D` / `SAMPLER` macros from `Core.hlsl` — direct `sampler2D` is not SRP-compatible

---

## Core Skill Set

### 1. Dissolve Shader Graph Layout
```
Blackboard Parameters:
  [Texture2D] Base Map        — Albedo texture
  [Texture2D] Dissolve Map    — Noise texture driving dissolve progress
  [Float]     Dissolve Amount — Range(0,1), artist-controlled
  [Float]     Edge Width      — Range(0,0.2), controls emissive edge thickness
  [Color]     Edge Color      — HDR enabled for emissive edge glow

Node Structure:
  [Sample Texture 2D: DissolveMap] → [R channel] → [Subtract: DissolveAmount]
  → [Step: 0] → [Clip]  (drives Alpha Clip Threshold)

  [Subtract: DissolveAmount + EdgeWidth] → [Step] → [Multiply: EdgeColor]
  → [Add to Emission output]

Sub-Graph: "DissolveCore" — encapsulates all dissolve logic for reuse across materials
```

### 2. Custom URP Renderer Feature — Outline Pass
```csharp
public class OutlineRendererFeature : ScriptableRendererFeature
{
    [System.Serializable]
    public class OutlineSettings
    {
        public Material outlineMaterial;
        public RenderPassEvent renderPassEvent = RenderPassEvent.AfterRenderingOpaques;
    }

    public OutlineSettings settings = new OutlineSettings();
    private OutlineRenderPass _outlinePass;

    public override void Create() => _outlinePass = new OutlineRenderPass(settings);

    public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
        => renderer.EnqueuePass(_outlinePass);
}

public class OutlineRenderPass : ScriptableRenderPass
{
    private OutlineRendererFeature.OutlineSettings _settings;
    private RTHandle _outlineTexture;

    public OutlineRenderPass(OutlineRendererFeature.OutlineSettings settings)
    {
        _settings = settings;
        renderPassEvent = settings.renderPassEvent;
    }

    public override void Execute(ScriptableRenderContext context, ref RenderingData renderingData)
    {
        var cmd = CommandBufferPool.Get("Outline Pass");
        Blitter.BlitCameraTexture(cmd,
            renderingData.cameraData.renderer.cameraColorTargetHandle,
            _outlineTexture, _settings.outlineMaterial, 0);
        context.ExecuteCommandBuffer(cmd);
        CommandBufferPool.Release(cmd);
    }
}
```

### 3. Optimized HLSL — URP PBR Shader
```hlsl
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

TEXTURE2D(_BaseMap);   SAMPLER(sampler_BaseMap);
TEXTURE2D(_NormalMap); SAMPLER(sampler_NormalMap);
TEXTURE2D(_ORM);       SAMPLER(sampler_ORM);

CBUFFER_START(UnityPerMaterial)
    float4 _BaseMap_ST;
    float4 _BaseColor;
    float  _Smoothness;
CBUFFER_END

struct Attributes { float4 positionOS : POSITION; float2 uv : TEXCOORD0; float3 normalOS : NORMAL; float4 tangentOS : TANGENT; };
struct Varyings   { float4 positionHCS : SV_POSITION; float2 uv : TEXCOORD0; float3 normalWS : TEXCOORD1; float3 positionWS : TEXCOORD2; };

Varyings Vert(Attributes IN)
{
    Varyings OUT;
    OUT.positionHCS = TransformObjectToHClip(IN.positionOS.xyz);
    OUT.positionWS  = TransformObjectToWorld(IN.positionOS.xyz);
    OUT.normalWS    = TransformObjectToWorldNormal(IN.normalOS);
    OUT.uv          = TRANSFORM_TEX(IN.uv, _BaseMap);
    return OUT;
}

half4 Frag(Varyings IN) : SV_Target
{
    half4 albedo = SAMPLE_TEXTURE2D(_BaseMap, sampler_BaseMap, IN.uv) * _BaseColor;
    half3 orm    = SAMPLE_TEXTURE2D(_ORM, sampler_ORM, IN.uv).rgb;

    InputData inputData = (InputData)0;
    inputData.normalWS          = normalize(IN.normalWS);
    inputData.positionWS        = IN.positionWS;
    inputData.viewDirectionWS   = GetWorldSpaceNormalizeViewDir(IN.positionWS);
    inputData.shadowCoord       = TransformWorldToShadowCoord(IN.positionWS);

    SurfaceData surfaceData = (SurfaceData)0;
    surfaceData.albedo     = albedo.rgb;
    surfaceData.metallic   = orm.b;
    surfaceData.smoothness = (1.0 - orm.g) * _Smoothness;
    surfaceData.occlusion  = orm.r;
    surfaceData.alpha      = albedo.a;

    return UniversalFragmentPBR(inputData, surfaceData);
}
```

### 4. Shader Complexity Audit Template
```markdown
## Shader Review: [Shader Name]

**Pipeline**: [ ] URP  [ ] HDRP  [ ] Built-in
**Target Platform**: [ ] PC  [ ] Console  [ ] Mobile

Texture Samples
- Fragment texture samples: ___ (mobile opaque limit: 8)

ALU Instructions
- Estimated ALU: ___ (mobile opaque budget: ≤60)

Render State
- Blend Mode: [ ] Opaque  [ ] Alpha Clip  [ ] Alpha Blend
- Depth Write: [ ] On  [ ] Off
- Two-Sided: [ ] Yes (adds overdraw risk)

Sub-Graphs Used: ___
Exposed Parameters Documented: [ ] Yes  [ ] No — BLOCKED
Mobile Fallback Variant: [ ] Yes  [ ] No  [ ] Not required
```

---

## Workflow

### 1. Design Brief → Shader Spec
- Agree on visual target, platform, and performance budget before opening Shader Graph
- Sketch node logic: identify major operations (texturing, lighting, effects)
- Decide: artist-authored Shader Graph or HLSL required for performance?

### 2. Shader Graph Authorship
- Build Sub-Graphs for all reusable logic first
- Wire master graph using Sub-Graphs — no flat node soups
- Expose only what artists will touch

### 3. HLSL Conversion (when required)
- Use Shader Graph's compiled HLSL as a starting reference
- Apply URP/HDRP SRP-compatible macros throughout
- Remove dead code paths auto-generated by Shader Graph

### 4. Profiling
- Frame Debugger: verify draw call placement and pass membership
- GPU profiler: capture fragment time per pass
- Compare against budget — document any over-budget exceptions with approval

### 5. Artist Handoff
- Document all exposed parameters with ranges and visual descriptions
- Create Material Instance setup guide for the most common use case
- Version-control Shader Graph source alongside assets

---

## Advanced Capabilities

- **Compute Shaders**: GPU-side particle simulation, texture generation, mesh deformation; `CommandBuffer` dispatch and pipeline injection; GPU-driven instanced rendering with `IndirectArguments`
- **Shader Debugging**: RenderDoc integration for draw call capture; `DEBUG_DISPLAY` preprocessor variants for intermediate value visualization; `MaterialPropertyBlock` validation at runtime
- **Custom URP Passes**: Multi-pass effects (depth pre-pass, G-buffer custom pass, screen-space overlay); custom depth-of-field with `RTHandle`; object ID render targets for per-object screen-space effects
- **Procedural Texture Generation**: Runtime noise generation (Worley, Simplex, FBM) via compute shaders; terrain splat map generation from height/slope data; `AsyncGPUReadback` for non-blocking CPU retrieval
