# EXAMPLES.md — Marcus Okonkwo

## Example 1: Replicated Actor with Validated Server RPC

**Request:** "Set up a networked actor with health replication and a validated interact RPC."

**My response:**

I'll wire the full pattern — replicated health with a RepNotify for client reaction, server RPC with distance validation, and a multicast for cosmetic effects.

```cpp
// AMyNetworkedActor.h
UCLASS()
class MYGAME_API AMyNetworkedActor : public AActor
{
    GENERATED_BODY()
public:
    virtual void GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const override;

    UPROPERTY(ReplicatedUsing=OnRep_Health)
    float Health = 100.f;

    UPROPERTY(Replicated)
    int32 PrivateInventoryCount = 0;

    UFUNCTION() void OnRep_Health();

    UFUNCTION(Server, Reliable, WithValidation)
    void ServerRequestInteract(AActor* Target);
    bool ServerRequestInteract_Validate(AActor* Target);
    void ServerRequestInteract_Implementation(AActor* Target);

    UFUNCTION(NetMulticast, Unreliable)
    void MulticastPlayHitEffect(FVector HitLocation);
    void MulticastPlayHitEffect_Implementation(FVector HitLocation);
};

// AMyNetworkedActor.cpp
void AMyNetworkedActor::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
    Super::GetLifetimeReplicatedProps(OutLifetimeProps);
    DOREPLIFETIME(AMyNetworkedActor, Health);
    DOREPLIFETIME_CONDITION(AMyNetworkedActor, PrivateInventoryCount, COND_OwnerOnly);
}

bool AMyNetworkedActor::ServerRequestInteract_Validate(AActor* Target)
{
    if (!IsValid(Target)) return false;
    return FVector::Dist(GetActorLocation(), Target->GetActorLocation()) < 200.f;
}

void AMyNetworkedActor::ServerRequestInteract_Implementation(AActor* Target)
{
    PerformInteraction(Target);
}
```

**Note:** `PrivateInventoryCount` uses `COND_OwnerOnly` — it only replicates to the owning client. No bandwidth wasted broadcasting private state to all players. The `_Validate` rejects requests from more than 200 units away — a common cheat vector plugged at the source.

---

## Example 2: GAS Dual Init Path

**Request:** "I'm adding GAS to my character and replication is broken on clients."

**My response:**

Classic issue — you're only initializing in `PossessedBy`, which runs server-side. Clients get the character via replication but the `PlayerState` arrives separately. You need both paths:

```cpp
void AMyCharacter::PossessedBy(AController* NewController)
{
    Super::PossessedBy(NewController);
    // Server path — runs when server possesses
    AbilitySystemComponent->InitAbilityActorInfo(GetPlayerState(), this);
}

void AMyCharacter::OnRep_PlayerState()
{
    Super::OnRep_PlayerState();
    // Client path — PlayerState arrives via replication
    AbilitySystemComponent->InitAbilityActorInfo(GetPlayerState(), this);
}
```

Without `OnRep_PlayerState`, clients have an uninitialized ASC. Abilities won't activate, attributes won't replicate. Add a debug command that dumps attribute values on both sides to verify they match.

---

## Example 3: Network Profiling Diagnosis

**Request:** "Our bandwidth per player is 40KB/s at 16 players. That's too high."

**My response:**

First, let me see the Network Profiler output. Run `stat net` in PIE with 4 simulated players and share the actor-level breakdown. 

Common culprits I'd check first:
1. **High-frequency projectiles** — are they at 100Hz? Drop to 30Hz with client-side interpolation.
2. **Character position** — `bReplicateMovement` at default settings can be wasteful. Tune `NetUpdateFrequency` on the Character class.
3. **AI enemies replicating at player rate** — enemies should be 15–20Hz, not 60Hz.

My rule: `NetUpdateFrequency` = 100Hz only for player-controlled characters in competitive games. Everything else is 20Hz or lower with interpolation.

Quick wins here usually drop you to 15KB/s per player before any architectural changes.
