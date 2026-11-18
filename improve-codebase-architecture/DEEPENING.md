# Deepening

In a Unity/Mirror context a **module** is usually a `MonoBehaviour`, `NetworkBehaviour`, plain C# class, or a small set of these that collaborate; its **interface** is the public API plus the networked surface (Commands, ClientRpcs/TargetRpcs, and `[SyncVar]`s + hooks it reads/writes).

In this data-driven project a module's interface also includes its `ScriptableObject` schema (`UnitDataSO`, `WaveDataSO`, …): content authors and their existing assets are callers too. Changing an SO class's fields is an interface change with one call site per authored asset.

## Dependency categories

When assessing a candidate for deepening, classify its dependencies. The category determines how the deepened module is structured and verified across its seam.

### 1. In-process

Pure computation, in-memory state, no I/O. Always deepenable — merge the modules and verify through the new interface directly. No adapter needed. Mind the cost: merging plain C# classes is free; merging `MonoBehaviour`s requires serialized data migration — weigh that in the candidate write-up.

### 2. Engine and data dependencies

Dependencies on Unity engine services (time, randomness, input, physics queries) and config from `ScriptableObject`s. The SO config seam is real — each authored data asset is an adapter, and swapping assets is how behaviour varies. Engine services have no second adapter in this project (no test suite, no replay system), so they are not real seams — call them directly.

### 3. Networked but owned (server/client seam)

The Mirror boundary between your own server and client code — the most important seam in this project, because the host is the single source of truth. The authoritative logic should live in one deep module guarded by `[Server]`/`isServer`; the network transport (`[Command]`, `[ClientRpc]`/`[TargetRpc]`, `[SyncVar]` replication) is the adapter that carries intent across the wire.

Keep the deep module's decision logic independent of *how* the request arrived. The `[Command]` follows the Commands rules in [AGENTS.md](../../../AGENTS.md) (Mirror Networking Patterns), then delegates to a `[Server]` method that applies the authoritative state change, with the pure decision rules factored into plain C# so they can be exercised without a live server.

Recommendation shape: *"`[Server]` entry point over plain-C# decision rules; Command is a thin validation adapter."*

### 4. True external (wrap behind a port)

Third-party services you don't control — Steam, Unity Gaming Services, analytics, ads, IAP, cloud saves. The deepened module takes the external dependency as an injected port (a C# interface you define); production wires the real SDK adapter, and any offline/dev build can provide a fake adapter. Never let the third-party SDK type leak into the deep module's interface — wrap it.

## Seam discipline

- **The server/client seam in category 3 is always real:** host and remote-client invocation are two genuine adapters.
- **Internal seams vs external seams.** A deep module can have internal seams (private to its implementation — a swapped targeting strategy, an `ObjectPool<T>` behind a spawn method) as well as the external seam at its interface (its public API and networked surface). Don't expose internal seams through the interface just because the implementation uses them. In particular, don't make a `[SyncVar]` or a Command/Rpc public solely to enable substitution — that widens the networked interface unnecessarily.

## Sanctioned seams (do not flag)

These patterns look shallow under the deletion test but are intentional. If one genuinely accumulates friction, raise it to the user in prose — never surface it as a deepening candidate.

- **The UI→server pipeline.** ViewModel → SO event → `[Command]` → validation → `[SyncVar]` hook → SO event → ViewModel. Each link is deliberately thin. Apply the deletion test to the modules at the *ends* (ViewModel, server-side decision logic), not the links.
- **Generated `GameEvent<T>` wrappers** under `Assets/Scripts/Util/GameEvents/`. Sanctioned generated infrastructure — never flag as shallow or propose consolidating.

## Verification strategy: through the interface, not the internals

- **Verify observable outcomes, not internal state.** Confirm what a client *sees* — the synced value, the spawned object, the rejected action. Client-observable outcomes lag server decisions by the SyncVar batching interval in [AGENTS.md](../../../AGENTS.md) (Mirror Networking Patterns).
- **Respect hook timing.** SyncVar hooks fire synchronously on the server at assignment — set all related state *before* assigning a hooked field to avoid re-entrancy.
- **Prefer logic you can exercise without a live session.** Keep decision rules in plain C# — `[Server]` methods no-op when `NetworkServer.active` is false. Drive the plain C# part directly in the Editor; use ParrelSync clones to confirm the `[Server]` entry point and replication.
- **Keep the interface small enough to review by reading.**
