# Impact Areas

The areas a design change can affect. Skills that assess or compare designs link here; each skill states how it applies them.

- **Serialized data:** renamed or retyped `[SerializeField]` fields (need `FormerlySerializedAs`), `.meta` GUIDs referenced by scenes and prefabs, string IDs on ScriptableObject assets, Unity-Weld bindings matched by property name, methods wired through UnityEvents (`m_MethodName` in prefabs, scenes, and `.asset` files), and `EventData` struct shapes.
- **Network roles:** behavior on the host, a remote client, and a late joiner; SyncVar hook ordering; latency from `syncInterval` batching.
- **Security and trust:** Command validation against cheating clients, and keeping state mutation server-authoritative.
- **Performance:** added per-frame work, GC allocations, pooling of frequently spawned objects, and network bandwidth from SyncVar and RPC traffic.
- **Extensibility:** whether new units, abilities, waves, or UI panels can be added through ScriptableObject data without modifying existing code.
- **Game design:** consistency with the rules and vocabulary in the [game-design](../game-design/SKILL.md) skill.
- **Verification:** which manual Play Mode check confirms the behavior, including host and client instances via ParrelSync when networking is involved.
