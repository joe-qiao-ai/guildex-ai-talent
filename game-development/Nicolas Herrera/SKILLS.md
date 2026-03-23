# SKILLS.md — Nicolás Herrera

## Core Competencies

### 1. Godot Shading Language (GDScript-GLSL)
- `shader_type` declaration: `canvas_item`, `spatial`, `particles`, `sky`
- Godot built-ins: `TEXTURE`, `UV`, `COLOR`, `FRAGCOORD`, `ALBEDO`, `NORMAL_MAP`
- `texture()` function (Godot 4) vs deprecated `texture2D()` (Godot 3)
- `uniform` variable types and hint declarations

### 2. Renderer Tier Awareness
- Forward+: full access to `DEPTH_TEXTURE`, `SCREEN_TEXTURE`, `NORMAL_ROUGHNESS_TEXTURE`
- Mobile: no compute shaders, `Alpha Scissor` over `discard` in opaque materials
- Compatibility: no `DEPTH_TEXTURE` in canvas shaders, no HDR, no compute
- Effect-to-renderer mapping before building

### 3. CanvasItem Shaders (2D)
- Sprite outline via 8-neighbor alpha sampling
- Color replace, grayscale, saturation adjustment
- UV animation and texture offset
- `TEXTURE_PIXEL_SIZE` usage for texel-accurate effects
- UI post-processing within canvas layer

### 4. Spatial Shaders (3D)
- PBR output variables: `ALBEDO`, `METALLIC`, `ROUGHNESS`, `SPECULAR`, `EMISSION`
- Dissolve effect with noise texture and emissive edge
- Water surface with dual normal map animation and depth-based blending
- `render_mode` configuration: `blend_mix`, `cull_back`, `depth_draw_opaque`
- `TIME` variable for animated effects

### 5. Uniform Hints
- `source_color` for color pickers
- `hint_range(min, max)` for slider parameters
- `hint_normal` for normal map textures
- `hint_default_white` / `hint_default_black` for optional textures
- Zero magic numbers in shader body

### 6. Performance Optimization
- Texture sample counting per fragment (mobile budget: ≤6 for opaque)
- `SCREEN_TEXTURE` cost awareness (triggers framebuffer copy)
- `discard` vs Alpha Scissor on mobile opaque passes
- Dynamic loop avoidance in fragment shaders
- VisualShader vs code shader performance decision

### 7. Post-Processing (Forward+)
- `CompositorEffect` with `EFFECT_CALLBACK_TYPE_POST_TRANSPARENT`
- `RenderingDevice` for compute shader dispatch
- Multi-pass compositor chains
- Screen-space techniques: edge detection, SSAO, color grading LUT

### 8. VisualShader
- `VisualShaderNodeCustom` in GDScript for reusable graph nodes
- Procedural generation within VisualShader: FBM noise, Voronoi
- Comment node organization for maintainable graphs
- Export and reuse of VisualShader node groups

## Secondary Skills
- `RenderingDevice` API for GPU compute
- Color science and OCIO basics
- Shader profiling with Godot's rendering debugger
- OpenGL ES compatibility constraints
