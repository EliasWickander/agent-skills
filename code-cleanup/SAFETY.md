# Safety

All Unity assets in this project are serialized as text YAML, so they can be grepped.

## Proving something is unused

A C# search alone is not enough. Before reporting a symbol as dead, check each of these. If any is inconclusive, keep the finding at Judgment tier and say what couldn't be confirmed.

1. **C# references** — grep the name across `Assets/Scripts/` and `Assets/3rdParty/` (toolkit code may call through interfaces or base classes).
2. **String references** — `nameof(...)` (SyncVar `hook =`), `Invoke("...")`, `SendMessage`, `Animator` events.
3. **Unity-Weld bindings** — prefabs and scenes reference ViewModel members as `viewModelPropertyName: ClassName.Member` and `viewModelMethodName: ClassName.Method`. Grep `*.prefab` and `*.unity`.
4. **UnityEvent wiring** — `m_MethodName: MethodName` in `*.prefab`, `*.unity`, `*.asset`. Method names aren't unique (many are `Trigger`), so confirm via the entry's `m_Target` which script it points at.
5. **Engine and Mirror entry points** — Unity messages (`Awake`, `Update`, `OnValidate`, `OnTrigger*`, …), Mirror overrides (`OnStartServer`, `OnStartClient`, …), and `[Command]`/`[ClientRpc]`/`[TargetRpc]` methods are called by the engine or weaver. Never dead by grep logic alone.
6. **Interface implementations and overrides** — used if the interface or base member is used.
7. **Script and SO types** — read the GUID from the script's `.cs.meta` (`guid:` line), then grep `guid: <that guid>` across `*.prefab`, `*.unity`, `*.asset`. Any match means it's in use.
8. **SO assets** (events, data) — same GUID search using the asset's `.meta`. Assets under `Assets/Resources/` can also be loaded by path, so grep the path string too.
9. **Editor usage** — `Editor/` folders, `[MenuItem]`, `[CustomEditor]`, `[CustomPropertyDrawer]`.

## Renaming safely

Renames that break assets fail at runtime or silently, not at compile time.

- **Serialized fields** — add `[FormerlySerializedAs("m_oldName")]`. It must stay until every referencing asset has been re-serialized; tell the user.
- **`MonoBehaviour` / `ScriptableObject` classes** — the file name must match the class. Rename the `.cs` and its `.cs.meta` together (use the move tool on both). Never delete and recreate a script.
- **`[Binding]` members** — update every matching `viewModelPropertyName` / `viewModelMethodName` string in prefabs and scenes. Renaming the ViewModel class changes the `ClassName.` prefix in all of them.
- **UnityEvent targets** — update matching `m_MethodName` entries whose `m_Target` is the renamed script.
- **SyncVar hooks** — use `nameof` so the compiler catches mismatches.
- **Mirror members** — renaming Commands, Rpcs, or SyncVars changes weaver-generated hashes. That's fine (no protocol compatibility needed), but host and clients must run the same build.
- **Assets** — check `Resources.Load` paths for anything under `Assets/Resources/`.

## Editing YAML assets

- Only do plain string replacements of binding names and `m_MethodName` values. Never hand-edit `fileID`s, GUIDs, or structure; ask the user to do those in the Editor.
- List every asset file touched in the batch summary.

## Verification

After each batch:

1. **Compile** — check diagnostics on every touched file.
2. **Editor** — after Unity reimports: no new Console errors, no "Missing (Mono Script)" or missing references on the touched prefabs and scenes.
3. **Play-test list** — name the specific flows that exercise the changed code (e.g. "lobby: toggle ready, start game"). Use the manual-check setups in `AGENTS.md` (Verification and Editor Work).
4. **Behavior change findings** — describe the observable difference the user should now see.
