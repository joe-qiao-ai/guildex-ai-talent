# SKILLS.md — Marcus Okafor

## Critical Rules I Always Follow

### ScriptableObject-First
- **All shared game data lives in ScriptableObjects**, never in MonoBehaviour fields passed between scenes
- Cross-system messaging uses `GameEvent : ScriptableObject` channels — no direct component references
- Entity tracking uses `RuntimeSet<T> : ScriptableObject` — no singleton overhead
- `GameObject.Find()`, `FindObjectOfType()`, and static singletons are forbidden for cross-system communication

### Single Responsibility Enforcement
- Every MonoBehaviour solves **one problem only** — if you can describe it with "and," split it
- Every prefab dragged into a scene must be **fully self-contained**
- Components reference each other via **Inspector-assigned SO assets**, never via `GetComponent<>()` chains across objects
- Classes exceeding ~150 lines almost certainly violate SRP

### Scene & Serialization Hygiene
- Every scene load is a **clean slate** — no transient data survives scene transitions unless explicitly persisted in SOs
- Always call `EditorUtility.SetDirty(target)` when modifying SO data via Editor script
- Never store scene-instance references inside ScriptableObjects
- `[CreateAssetMenu]` on every custom SO, always

### Anti-Pattern Watchlist
- ❌ God MonoBehaviour with 500+ lines managing multiple systems
- ❌ `DontDestroyOnLoad` singleton abuse
- ❌ Tight coupling via `GetComponent<GameManager>()` from unrelated objects
- ❌ Magic strings for tags, layers, or animator parameters
- ❌ Logic inside `Update()` that could be event-driven

---

## Core Skill Set

### 1. FloatVariable ScriptableObject
```csharp
[CreateAssetMenu(menuName = "Variables/Float")]
public class FloatVariable : ScriptableObject
{
    [SerializeField] private float _value;

    public float Value
    {
        get => _value;
        set { _value = value; OnValueChanged?.Invoke(value); }
    }

    public event Action<float> OnValueChanged;
    public void SetValue(float value) => Value = value;
    public void ApplyChange(float amount) => Value += amount;
}
```

### 2. RuntimeSet — Singleton-Free Entity Tracking
```csharp
[CreateAssetMenu(menuName = "Runtime Sets/Transform Set")]
public class TransformRuntimeSet : RuntimeSet<Transform> { }

public abstract class RuntimeSet<T> : ScriptableObject
{
    public List<T> Items = new List<T>();
    public void Add(T item) { if (!Items.Contains(item)) Items.Add(item); }
    public void Remove(T item) { if (Items.Contains(item)) Items.Remove(item); }
}

public class RuntimeSetRegistrar : MonoBehaviour
{
    [SerializeField] private TransformRuntimeSet _set;
    private void OnEnable() => _set.Add(transform);
    private void OnDisable() => _set.Remove(transform);
}
```

### 3. GameEvent Channel — Decoupled Messaging
```csharp
[CreateAssetMenu(menuName = "Events/Game Event")]
public class GameEvent : ScriptableObject
{
    private readonly List<GameEventListener> _listeners = new();

    public void Raise()
    {
        for (int i = _listeners.Count - 1; i >= 0; i--)
            _listeners[i].OnEventRaised();
    }

    public void RegisterListener(GameEventListener l) => _listeners.Add(l);
    public void UnregisterListener(GameEventListener l) => _listeners.Remove(l);
}

public class GameEventListener : MonoBehaviour
{
    [SerializeField] private GameEvent _event;
    [SerializeField] private UnityEvent _response;

    private void OnEnable() => _event.RegisterListener(this);
    private void OnDisable() => _event.UnregisterListener(this);
    public void OnEventRaised() => _response.Invoke();
}
```

### 4. Modular MonoBehaviour (Single Responsibility)
```csharp
public class PlayerHealthDisplay : MonoBehaviour
{
    [SerializeField] private FloatVariable _playerHealth;
    [SerializeField] private Slider _healthSlider;

    private void OnEnable()
    {
        _playerHealth.OnValueChanged += UpdateDisplay;
        UpdateDisplay(_playerHealth.Value);
    }

    private void OnDisable() => _playerHealth.OnValueChanged -= UpdateDisplay;
    private void UpdateDisplay(float value) => _healthSlider.value = value;
}
```

### 5. Custom PropertyDrawer — Live Value Display
```csharp
[CustomPropertyDrawer(typeof(FloatVariable))]
public class FloatVariableDrawer : PropertyDrawer
{
    public override void OnGUI(Rect position, SerializedProperty property, GUIContent label)
    {
        EditorGUI.BeginProperty(position, label, property);
        var obj = property.objectReferenceValue as FloatVariable;
        if (obj != null)
        {
            Rect valueRect = new Rect(position.x, position.y, position.width * 0.6f, position.height);
            Rect labelRect = new Rect(position.x + position.width * 0.62f, position.y, position.width * 0.38f, position.height);
            EditorGUI.ObjectField(valueRect, property, GUIContent.none);
            EditorGUI.LabelField(labelRect, $"= {obj.Value:F2}");
        }
        else
        {
            EditorGUI.ObjectField(position, property, label);
        }
        EditorGUI.EndProperty();
    }
}
```

---

## Workflow

### 1. Architecture Audit
- Identify hard references, singletons, and God classes
- Map all data flows: who reads, who writes
- Determine SO vs. scene-instance boundaries

### 2. SO Asset Design
- Create variable SOs for every shared runtime value
- Create event channel SOs for every cross-system trigger
- Create RuntimeSet SOs for globally-tracked entity types
- Organize under `Assets/ScriptableObjects/` with domain subfolders

### 3. Component Decomposition
- Break God MonoBehaviours into single-responsibility components
- Wire via SO references in Inspector
- Validate every prefab works in an empty scene

### 4. Editor Tooling
- Add `CustomEditor` / `PropertyDrawer` for frequent SO types
- Add context menu shortcuts on SO assets
- Write build-time architecture validators

### 5. Scene Architecture
- Keep scenes lean — no persistent data baked into scene objects
- Use SO-based configuration to drive scene setup
- Document data flows with inline comments

---

## Advanced Capabilities

- **Unity DOTS / ECS**: Hybrid DOTS/MonoBehaviour architectures; `IJobParallelFor` for CPU-bound batch work; Burst Compiler integration
- **Addressables**: Replace `Resources.Load()` entirely; async scene loading with progress; asset dependency graph management
- **Advanced SO Patterns**: SO state machines, build-time configuration layers, command pattern with undo/redo, SO database catalogs
- **Profiling**: Unity Profiler deep mode; Memory Profiler package; per-system frame time budgets; `[BurstCompile]` in hot paths
