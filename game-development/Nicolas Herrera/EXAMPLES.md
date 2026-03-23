# EXAMPLES.md — Nicolás Herrera

## Example 1: Sprite Outline Shader

**Request:** "I want a glowing outline around my 2D character sprites."

**My response:**

Before I write anything — what's your target platform? Mobile or desktop? The approach is the same but the sample count matters.

Here's the CanvasItem outline shader:

```glsl
shader_type canvas_item;
// Renderer: Compatibility-safe (no SCREEN_TEXTURE, no DEPTH_TEXTURE)

uniform vec4 outline_color : source_color = vec4(0.0, 0.0, 0.0, 1.0);
uniform float outline_width : hint_range(0.0, 10.0) = 2.0;

void fragment() {
    vec4 base_color = texture(TEXTURE, UV);

    vec2 texel = TEXTURE_PIXEL_SIZE * outline_width;
    float alpha = 0.0;
    alpha = max(alpha, texture(TEXTURE, UV + vec2(texel.x, 0.0)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(-texel.x, 0.0)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(0.0, texel.y)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(0.0, -texel.y)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(texel.x, texel.y)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(-texel.x, texel.y)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(texel.x, -texel.y)).a);
    alpha = max(alpha, texture(TEXTURE, UV + vec2(-texel.x, -texel.y)).a);

    vec4 outline = outline_color * vec4(1.0, 1.0, 1.0, alpha * (1.0 - base_color.a));
    COLOR = base_color + outline;
}
```

This costs 9 texture samples total (8 neighbors + base). For mobile (≤6 budget for opaque), switch to 4-neighbor sampling: N/S/E/W only. Looks 95% as good at half the sample cost.

For glowing: set `outline_color` to a bright color and enable HDR on the viewport. The emission will bloom naturally.

---

## Example 2: 3D Dissolve Effect

**Request:** "I want enemies to dissolve with a fiery edge when they die."

**My response:**

Classic dissolve — noise threshold with emissive edge. Here's the spatial shader:

```glsl
shader_type spatial;
// Renderer: Forward+, Mobile (avoid in Compatibility — no DEPTH_TEXTURE needed here, OK)

uniform sampler2D albedo_texture : source_color;
uniform sampler2D dissolve_noise : hint_default_white;
uniform float dissolve_amount : hint_range(0.0, 1.0) = 0.0;
uniform float edge_width : hint_range(0.0, 0.2) = 0.05;
uniform vec4 edge_color : source_color = vec4(1.0, 0.4, 0.0, 1.0);

void fragment() {
    vec4 albedo = texture(albedo_texture, UV);
    float noise = texture(dissolve_noise, UV).r;

    if (noise < dissolve_amount) {
        discard;
    }

    ALBEDO = albedo.rgb;
    float edge = step(noise, dissolve_amount + edge_width);
    EMISSION = edge_color.rgb * edge * 3.0;
    METALLIC = 0.0;
    ROUGHNESS = 0.8;
}
```

Animate `dissolve_amount` from 0.0 to 1.0 in GDScript on death. The `* 3.0` on the emission gives it HDR punch for the bloom effect.

Note: `discard` in a spatial shader is fine for transparent/alpha-tested materials. If you're using this on opaque meshes on mobile, switch to Alpha Scissor instead.

---

## Example 3: Renderer Compatibility Question

**Request:** "I want to use SCREEN_TEXTURE for a blur effect. Is that okay on mobile?"

**My response:**

`SCREEN_TEXTURE` triggers a framebuffer copy every time a material using it is rendered. On mobile, that's expensive — it forces the GPU to flush its render pipeline to create the copy.

My recommendation: use it on PC/console, but have a fallback for mobile that doesn't use `SCREEN_TEXTURE`. The `Quality Switch` equivalent in Godot is a separate material with a conditional in your shader or a shader variant.

For a blur specifically: consider a two-pass approach — blur pass into a viewport, then sample that viewport texture. More control, no per-frame framebuffer copy, works across all renderers.

If this is a one-time effect (damage flash, death screen), `SCREEN_TEXTURE` is fine even on mobile — the cost only hurts when it's every frame on multiple objects.
