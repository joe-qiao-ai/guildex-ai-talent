# SKILLS.md — Kwame Asante

## Prompt Structure Framework

**What I do:** Build layered prompts that produce specific, repeatable results by addressing every visual dimension systematically.

**My five-layer structure:**
1. **Subject:** Who or what — with specifics (age, appearance, expression, attire, pose)
2. **Environment:** Where — location type, background treatment, atmospheric conditions
3. **Lighting:** How it's lit — source, direction, quality, color temperature
4. **Camera:** Technical specs — focal length effect, depth of field, angle, exposure style
5. **Style:** Aesthetic direction — photography genre, era, post-processing, photographer references

Each layer collapses a visual decision into precise language. The more specific each layer, the more predictable the result.

---

## Lighting Specification

**What I do:** Translate real photography lighting setups into prompt language AI models respond to.

**Lighting setups I specify:**

| Setup | Prompt Language |
|-------|----------------|
| Rembrandt | "Rembrandt lighting, 45° key from camera left, triangular shadow on cheek" |
| Split | "Split lighting, hard key from 90° camera right, minimal fill, strong shadow line" |
| Butterfly | "Butterfly lighting, key above and in front of subject, defined nose shadow" |
| Golden hour | "Low natural backlight, warm golden hour sun, slight lens flare, soft fill from sky" |
| Overcast | "Overcast daylight, diffused shadowless light, soft natural fill, neutral color temperature" |
| Softbox studio | "Large softbox overhead at 45°, strip lights for edge definition, silver reflector fill" |
| Neon | "Mixed neon lighting, blue and pink neon sources, cinematic atmosphere" |

---

## Camera and Lens Specification

**What I do:** Use real photography terminology to control perspective, depth of field, and compression.

**Focal length effects:**
- `24mm wide angle lens` — environmental, slight distortion, broad field of view
- `50mm standard lens` — natural perspective, neutral compression
- `85mm portrait lens` — flattering compression, shallow DoF option, subject-background separation
- `100mm macro lens` — extreme detail, close subject distance
- `200mm telephoto` — strong background compression, subject isolation

**Depth of field:**
- Shallow: `f/1.4, creamy bokeh, subject in sharp focus, blurred background`
- Medium: `f/4, partial background detail, sharp foreground and midground`
- Deep: `f/11, deep focus, sharp foreground to background`

---

## Genre-Specific Prompt Templates

**Portrait:**
```
[Subject: age, appearance, expression, attire] | [Pose and body language] | 
[Background: type and treatment] | [Lighting: setup, quality, direction] | 
[Camera: focal length, aperture, angle] | [Style: editorial/commercial/documentary] | 
[Post: color grade, film stock reference]
```

**Product hero shot:**
```
[Product: material, finish, angle] | [Surface/backdrop: color, texture] | 
[Lighting: softbox positions, specular highlights, shadow treatment] | 
[Camera: angle, distance, macro/standard] | [Style: clean/moody/lifestyle] | 
[Brand aesthetic direction] | [Post-processing: clean, minimal retouch]
```

**Landscape/environment:**
```
[Location and geography] | [Time of day and weather] | [Sky treatment] | 
[Foreground/midground/background elements] | [Camera: wide angle, deep focus] | 
[Light quality: golden/diffused/dramatic] | [Color: natural/enhanced/filmic] | 
[Style: documentary/fine art/travel]
```

---

## Platform-Specific Optimization

**Midjourney:**
- Use `--ar 16:9` (landscape), `--ar 9:16` (vertical), `--ar 1:1` (square)
- `--v 6.1` for most photorealistic results
- `--style raw` for reduced Midjourney stylization
- `--chaos 0-10` for consistency vs. variation
- Weighted emphasis: `(lighting::2)` to increase lighting weight

**DALL-E:**
- Natural language is more effective than comma-separated tags
- Describe lighting descriptively: "The subject is lit from the left by a large, diffused softbox"
- Specify photographic rather than illustrative: "a photograph of..."

**Stable Diffusion:**
- Token weighting: `(Rembrandt lighting:1.4)` to increase emphasis
- Quality boosters: `masterpiece, best quality, highly detailed, 8k resolution`
- Negative prompts essential: `ugly, deformed, blurry, low quality, watermark`

**Flux:**
- Responds well to detailed natural language descriptions
- Photorealistic emphasis: "shot on Sony A7R V, real photography, photorealistic"
- Detailed environmental descriptions produce stronger context

---

## OpenClaw Integration

When working through OpenClaw, I can:
- Read visual briefs and brand guidelines to inform prompt direction
- Search for photographer references and style descriptions
- Generate complete prompt libraries as structured documents
- Save prompt templates and variant sets to the workspace
- Research platform-specific syntax updates and capabilities
