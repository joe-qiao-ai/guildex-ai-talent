# TESTS.md — Nicolás Herrera

## Test 1: Renderer Tier Selection

**Prompt:** "I want to use DEPTH_TEXTURE in my water shader. What do I need?"

**Expected behavior:**
- `DEPTH_TEXTURE` requires Forward+ or Mobile renderer
- Not available in Compatibility renderer
- Recommends documenting this in the shader header: `// Renderer: Forward+ / Mobile required`
- If the project targets Compatibility (e.g., old mobile hardware), needs a simplified water without depth blending

**Pass criteria:** Correct renderer requirement, recommends documentation.

---

## Test 2: Godot 3 vs 4 Syntax

**Prompt:** "I'm using `texture2D(TEXTURE, UV)` in my shader and getting weird errors."

**Expected behavior:**
- Clear answer: `texture2D()` is Godot 3 syntax
- Godot 4 uses `texture(TEXTURE, UV)` — different function name
- Explains: this may "work" in some contexts but fail silently in others — always use Godot 4 syntax
- Provides the correct version

**Pass criteria:** Identifies the syntax issue and provides the Godot 4 correct form.

---

## Test 3: Uniform Hint Missing

**Prompt:** "My color uniform looks like a plain number field in the Inspector instead of a color picker."

**Expected behavior:**
- Missing `: source_color` hint on the uniform declaration
- Correct: `uniform vec4 my_color : source_color = vec4(1.0, 1.0, 1.0, 1.0);`
- Explains the other hints: `hint_range(min, max)` for sliders, `hint_normal` for normal maps
- Notes: hints make shaders usable by artists without touching code

**Pass criteria:** Identifies `source_color` as the fix, mentions other hints.

---

## Test 4: Mobile Texture Sample Budget

**Prompt:** "My fragment shader has 12 texture samples. Is that okay?"

**Expected behavior:**
- Asks: what's the target platform?
- For mobile: 12 is well over budget — mobile budget is ≤6 for opaque materials
- For PC: 12 is acceptable, check the rendering profiler to confirm it's within frame time
- Offers optimization strategies: combine texture channels, reduce neighbor samples
- Does NOT say "it's fine" without asking about platform

**Pass criteria:** Asks platform, gives mobile-specific budget number, offers optimization.

---

## Test 5: discard in Opaque Mobile Shader

**Prompt:** "I'm using `discard` in my spatial shader and performance is bad on mobile."

**Expected behavior:**
- `discard` in opaque spatial shaders breaks mobile GPU early-depth rejection
- Fix: use Alpha Scissor (material property in Godot 4) instead of shader-level `discard`
- Alpha Scissor lets the GPU cull fragments early without breaking the pipeline
- Notes: `discard` is fine in transparent passes — the issue is specifically in opaque render mode

**Pass criteria:** Identifies the early-depth rejection issue, recommends Alpha Scissor.
