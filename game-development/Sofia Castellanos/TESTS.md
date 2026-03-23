# TESTS.md — Sofia Castellanos

## Test 1: Material Function vs Duplication

**Prompt:** "I have the same rock blending logic in 12 different materials. Is that okay?"

**Expected behavior:**
- Clear answer: No — extract it to a Material Function
- Explains the maintenance problem: change the blend in one, you must change all 12
- Explains the permutation problem: duplicated logic can cause inconsistent permutation counts
- Provides guidance on how to create the Material Function and reference it
- Does NOT accept "it works, so it's fine"

**Pass criteria:** Recommends Material Function, explains both maintenance and technical reasons.

---

## Test 2: Niagara GPU vs CPU Decision

**Prompt:** "I want a magic aura effect with 3,000 simultaneous particles around the player. GPU or CPU simulation?"

**Expected behavior:**
- Answer: GPU simulation — above 1,000 particles, GPU is the right choice
- Notes: GPU simulation doesn't support per-particle collision (use depth buffer collision instead)
- Asks about target platform — mobile has more GPU simulation restrictions
- Recommends building scalability presets: High (3,000), Medium (1,500), Low (500)
- Sets `Max Particle Count` explicitly — never unlimited

**Pass criteria:** Correct GPU recommendation with scalability preset requirement and collision caveat.

---

## Test 3: Instruction Count Audit

**Prompt:** "My material has 520 instructions and we're targeting console. Is that okay?"

**Expected behavior:**
- Console budget is approximately 400 instructions for standard materials
- 520 is over budget — needs optimization
- Asks what's in the material: how many texture samples, any expensive operations?
- Suggests checking which nodes are most expensive using the Shader Complexity viewport mode
- Does NOT say "it depends" without the platform context already given

**Pass criteria:** Identifies the overage, recommends profiling approach.

---

## Test 4: PCG Determinism

**Prompt:** "Every time we re-run our PCG forest graph, the trees are in different positions. Is that expected?"

**Expected behavior:**
- Answer: No — PCG graphs should be deterministic
- Common cause: using a random seed that isn't fixed, or time-based randomization
- Fix: ensure the PCG graph uses a fixed seed or deterministic input parameters
- Explains: same input parameters → same output, always — this is a design invariant of PCG

**Pass criteria:** Identifies determinism requirement and fixes the seed issue.

---

## Test 5: Scalability Presets Timing

**Prompt:** "We're going to add scalability presets for our Niagara systems after the game ships."

**Expected behavior:**
- Strong pushback: scalability presets must be designed alongside the effect, not added later
- Explains why: the Low preset changes the effect design — if Low looks bad, the base design needs revision
- The frame budget conversation happens at design time, not after ship
- Minimum viable: Low/Medium/High defined and tested on lowest target hardware before any effect ships

**Pass criteria:** Clear pushback with design-time requirement explained.
