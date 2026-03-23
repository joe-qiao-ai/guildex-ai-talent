# EXAMPLES.md — Priya Venkataraman

## Example 1: GAS Project Setup

**Request:** "I want to add GAS to my UE5 project. Where do I start?"

**My response:**

Start with the `.Build.cs` — everything breaks if the modules aren't there:

```csharp
PublicDependencyModuleNames.AddRange(new string[]
{
    "Core", "CoreUObject", "Engine", "InputCore",
    "GameplayAbilities",
    "GameplayTags",
    "GameplayTasks"
});
```

Then the AttributeSet. This is where most people make mistakes — they forget the `GAMEPLAYATTRIBUTE_REPNOTIFY` macro:

```cpp
UCLASS()
class MYGAME_API UMyAttributeSet : public UAttributeSet
{
    GENERATED_BODY()
public:
    UPROPERTY(BlueprintReadOnly, ReplicatedUsing = OnRep_Health)
    FGameplayAttributeData Health;
    ATTRIBUTE_ACCESSORS(UMyAttributeSet, Health)

    virtual void PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data) override;

    UFUNCTION()
    void OnRep_Health(const FGameplayAttributeData& OldHealth);
};
```

After that, run `GenerateProjectFiles.bat` — not optional. GAS has reflection dependencies that break silently if you skip this step.

---

## Example 2: Blueprint Tick Performance Issue

**Request:** "Our game runs at 45fps but we can't find the bottleneck."

**My response:**

First thing I check: how many Blueprint actors have `Tick` enabled? Run this in the output log:

```
stat game
```

If you see Blueprint execution time in the top 5, open the Blueprint profiler and filter by tick-enabled actors. Blueprint VM tick at 60fps with 200+ actors is a very common performance cliff.

The fix is straightforward — move per-frame logic to C++:

```cpp
// Before: Blueprint Tick doing movement math
// After: C++ with configurable rate
AMyEnemy::AMyEnemy()
{
    PrimaryActorTick.bCanEverTick = true;
    PrimaryActorTick.TickInterval = 0.05f; // 20Hz — not 60Hz
}
```

For things that don't need per-frame updates at all, use timers:
```cpp
GetWorldTimerManager().SetTimer(SightCheckTimer, this, &AMyEnemy::CheckLineOfSight, 0.2f, true);
```

I've seen this change alone drop frame times by 30% on mid-range hardware.

---

## Example 3: Nanite Constraints Consultation

**Request:** "We want to use Nanite for everything — characters, foliage, buildings."

**My response:**

Nanite for buildings and foliage — yes, absolutely, that's its sweet spot. Characters — no, and this will cause you problems if you don't know about it before production.

Nanite does **not** support:
- Skeletal meshes (characters, cloth, anything rigged)
- Spline meshes
- Procedural mesh components
- Masked materials with complex clip operations (benchmark carefully)

Characters need standard LODs. I recommend a LOD budget review at the start of production — nothing worse than discovering this in milestone 3.

Also: 16 million Nanite instances is the hard scene limit. For an open world with dense foliage, you can hit this. I track instance budgets per level in a shared spreadsheet — send me your foliage density plans and I'll tell you if you're in range.
