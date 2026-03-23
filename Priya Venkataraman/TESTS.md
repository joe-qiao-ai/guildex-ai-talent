# TESTS.md — Priya Venkataraman

## Test 1: Blueprint vs C++ Decision

**Prompt:** "Should I implement my enemy AI steering in Blueprint or C++?"

**Expected behavior:**
- Asks: is this per-frame logic? (Yes — steering runs every tick)
- Answers: C++, because Blueprint VM overhead for per-frame logic at scale is measurable
- Explains: steering involves `FVector` math and collision queries that have data type limitations in Blueprint
- Notes: expose the tuning parameters to Blueprint via `UPROPERTY(EditDefaultsOnly)` so designers can adjust values
- Does NOT just say "C++" without explaining the tradeoff

**Pass criteria:** Correct recommendation with performance reasoning and designer workflow consideration.

---

## Test 2: GAS Attribute Clamping

**Prompt:** "How do I make sure Health never goes below 0 or above MaxHealth in GAS?"

**Expected behavior:**
- `PostGameplayEffectExecute` is the right hook
- Clamp using `SetHealth(FMath::Clamp(...))` via the ATTRIBUTE_ACCESSORS macro setter
- Explains why direct attribute mutation outside GAS breaks replication
- Bonus: mentions `PreAttributeChange` for pre-clamp validation

**Pass criteria:** Uses `PostGameplayEffectExecute`, explains the GAS replication reason.

---

## Test 3: Memory Bug Diagnosis

**Prompt:** "My game crashes with 'object is pending kill' errors randomly during gameplay."

**Expected behavior:**
- Primary cause: `UObject*` stored without `UPROPERTY()` — GC collects it
- Secondary cause: `IsValid()` not checked before use (checking `!= nullptr` isn't enough)
- Provides fix: add `UPROPERTY()` to all UObject pointer members
- Provides fix: replace `!= nullptr` checks with `IsValid()`
- Mentions `TWeakObjectPtr` for non-owning references that shouldn't prevent GC

**Pass criteria:** Identifies both root causes with correct fixes.

---

## Test 4: Module Dependency Error

**Prompt:** "I added GAS and now I get link errors when building."

**Expected behavior:**
- Checks: are `GameplayAbilities`, `GameplayTags`, `GameplayTasks` in `.Build.cs`?
- Checks: did they run `GenerateProjectFiles.bat` after modifying `.Build.cs`?
- Checks: are there circular module dependencies?
- Walks through the exact `.Build.cs` configuration needed
- Does NOT guess at link error causes without the above checklist

**Pass criteria:** Asks for and works through the `.Build.cs` systematically.

---

## Test 5: Nanite vs LOD Decision

**Prompt:** "Our character artist wants to know: should we use Nanite for player characters?"

**Expected behavior:**
- Clear answer: No — Nanite doesn't support skeletal meshes
- Alternative: standard LOD chains with verified transition distances
- Explains what Nanite IS good for on the same project: weapons (when not equipped), environment props
- Does not hedge or say "it depends" without explanation

**Pass criteria:** Definitive answer with correct technical reasoning.
