# Cleanup Checklist

Rules come from `AGENTS.md` — this file says how to find violations, not what the rules are.

Hot-path code means `Update`/`LateUpdate`/`FixedUpdate` and anything called per unit per frame (targeting, movement, damage, ability ticks).

## 1. Convention violations

Check every section of `AGENTS.md`: C# Style & Structure, Events, Mirror Networking Patterns, UI & MVVM. Detection hints:

- **Forbidden constructs** — grep `namespace `, `#region`, `NotImplementedException`, `GameObject.Find`, `FindObjectOfType`, `FindFirstObjectByType`, `FindAnyObjectByType`, `transform.Find`, `StartCoroutine`, `IEnumerator`.
- **Expression-bodied methods** — method declarations ending in `) =>`. Properties are fine.
- **Implicit access modifiers** — fields and methods without `private`/`public`/`protected`/`internal`.
- **Inline attributes** — an attribute on the same line as its declaration, e.g. `[SerializeField] private`.
- **Braces** — braces around single-statement bodies, or a body on the same line as `if`/`for`/`foreach`/`while`.
- **Member order** — fields → properties → events → Unity lifecycle → Mirror callbacks → Commands/Rpcs → public → private.
- **Naming** — `m_`/`s_` prefixes, `SCREAMING_SNAKE_CASE` constants, `Async` suffix, `Cmd`/`Rpc`/`TargetRpc` prefixes, `DataSO`/`EventData`/`ViewModel` suffixes, boolean methods phrased as questions, event names (past tense vs `Request`, `Invoke`/`On` prefixes).
- **ScriptableObjects** — missing `[CreateAssetMenu]`; public fields instead of `[SerializeField]` + read-only property.
- **ViewModels** — missing `[Binding]` on class or members; setters that don't raise `OnPropertyChanged`; `PropertyChangedEventArgs` allocated per call instead of cached; references to `NetworkBehaviour`s or direct Command calls.
- **Structure** — file name ≠ class name; file in a folder that doesn't match its system.
- **`var`** — any use; `AGENTS.md` requires explicit types even when obvious.

## 2. Duplication

- Same logic in two or more places — a non-trivial expression or ~5+ lines. Extract into a shared private method, a pure function in `Math/`, or a utility in `Core/`.
- Repeated boilerplate: identical lookup chains, null guards, validation preambles in Commands.
- Parallel `switch`es over the same enum or ID in different files.
- The same constant or string ID defined or written as a literal in several files.
- Duplicate `EventData` structs with the same meaning.

Before merging, ask: *if one copy changes, must the other?* If not, it's coincidental duplication — leave it. For small snippets, wait for a third copy.

Duplication that exists because a responsibility has no home (several systems each re-implement part of a concept) is an architecture finding — escalate it.

## 3. Local refactoring opportunities

Read [REFACTORING.md](REFACTORING.md) for the in-scope catalog and the architecture escalation boundary.

## 4. Dead code and orphaned assets

Run the reference checks in [SAFETY.md](SAFETY.md) before reporting anything here.

- Unused private members, parameters, locals, and `using`s.
- Unused public types and members, or scripts that no scene, prefab, or `.asset` references by GUID.
- `[SerializeField]` fields never read in code.
- Commented-out code.
- Empty Unity messages (`Update`, `Start`, `OnEnable`, …) — empty `Update`s still cost a native→managed call per object per frame.
- `[FormerlySerializedAs]` attributes whose migration looks complete — report only; removal is safe only after every referencing asset is re-serialized.

## 5. Event wiring

- Every subscription has a matching unsubscription with the same method: `OnEnable`/`OnDisable` for `+=`, `RegisterListener`, `AddListener`. Subscriptions in `Awake`/`Start` need a matching removal in `OnDestroy`. Missing removals are **Behavior change** (leaks, double handling after re-enable).
- Lambdas or anonymous methods subscribed to events.
- C# `event`s used between unrelated systems — the convention calls for an SO event. Replacing one adds a seam, so escalate it.
- SO event payloads that aren't an `EventData` struct.
- Event asset names that don't follow `{PastTenseAction}Event` / `{Action}RequestEvent`.

## 6. Mirror hygiene

Fixes here are usually **Behavior change**.

- Game state changed outside `[Server]` methods or `isServer` guards, including SyncVar writes on clients.
- `[Command]`s that don't validate at the top, or that apply state before validating.
- `[Command(requiresAuthority = false)]` without sender validation.
- Commands declared on objects the player doesn't own.
- SyncVar hooks that aren't idempotent, or that aren't invoked manually in `OnStartClient`.
- Initialization in `Start` that Mirror callbacks or hooks depend on — move it to `Awake` or `EnsureInitialized`.
- `Instantiate`/`Destroy` on objects with `NetworkIdentity`, instead of `NetworkServer.Spawn`/`Destroy`/`UnSpawn`.
- `SyncList` fields reassigned instead of mutated.
- SyncVar custom structs that aren't `readonly struct` + `IEquatable<T>`.
- Polling synced state in `Update` to detect changes instead of reacting in hooks.

## 7. Hot-path hygiene

In hot-path code:

- `GetComponent*`, `Camera.main`, `FindObjectsOfType` — cache in `Awake`.
- LINQ, `new List`/arrays, string concatenation or interpolation, closures capturing locals, `params` calls, `foreach` over interface-typed collections (boxed enumerator).
- Frequent `Instantiate`/`Destroy` — use `ObjectPool<T>`.
- `tag ==` instead of `CompareTag`.
- Repeated null checks on serialized references — validate once in `OnValidate` or `Awake`.

State the hot-path impact in the finding (what runs how often, what allocation or call is removed).

## 8. Data-driven violations

- Numeric literals that are tunables (ranges, speeds, costs, timings, counts) — move to an SO field. Literals that are fixed facts get a named constant.
- Code that branches on specific unit, wave, or ability IDs, or hardcodes them as string literals.
- Enums that enumerate content (unit types, wave types) rather than mechanics.
- Game code writing to SO fields at runtime.
- Duplicate string IDs across SO assets.

## 9. Unity gotchas

Check against the Unity rules in `AGENTS.md` (Universal Conventions):

- `?.`, `??`, `??=`, `is null`, `is not null` on `UnityEngine.Object` subclasses (`MonoBehaviour`, `NetworkBehaviour`, `ScriptableObject`, `Component`, `GameObject`, `Transform`).
- Mutable `static` fields or events without a `[RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.SubsystemRegistration)]` reset method. Missing resets are **Behavior change**.
- `async void` methods outside event handlers.
- `Awaitable` calls missing `destroyCancellationToken`, or code after `await` touching `this`, `transform`, or serialized references without guarding against destruction.

## 10. Logging and comments

- `Debug.Log` left over from debugging, logs in hot paths, wrong severity (`Log` for what is a warning or error), inconsistent message formats.
- Comments that restate what the code does, describe callers, or list internal steps.
- Stale comments that no longer match the code.
- XML docs that only restate the method name.
- TODOs without enough context to act on.
