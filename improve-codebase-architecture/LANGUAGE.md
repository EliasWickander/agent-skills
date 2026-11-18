# Language

## Terms

**Module**
_Avoid_: unit, component, service.

**Interface**
_Avoid_: API, signature.

**Implementation**
Distinct from **Adapter**: a thing can be a small adapter with a large implementation (a pooled Mirror spawn handler over real spawn logic) or a large adapter with a small implementation (a `MonoBehaviour` wrapper around a pure-C# system, or an in-memory fake of a data source). Reach for "adapter" when the seam is the topic; "implementation" otherwise.

**Seam** _(from Michael Feathers)_
Choosing where to put the seam is its own design decision, distinct from what goes behind it. In this project, common seams are C# `interface`s, ScriptableObject event channels (`GameEvent<T>`), and `[SyncVar]` hooks.
_Avoid_: boundary (overloaded with DDD's bounded context); say **seam** or **interface**.

**Adapter**
Describes *role* (what slot it fills), not substance (what's inside). Examples here: a server-side SO-event listener that validates a request, or a Mirror spawn handler backed by an `ObjectPool<T>`.

## Principles

- **Depth is a property of the interface, not the implementation.** A deep module can be internally composed of small, swappable parts — they just aren't part of the interface.

## Relationships

- A **Module** has exactly one **Interface** (the surface it presents to callers and reviewers).

## Rejected framings

- **Depth as ratio of implementation-lines to interface-lines** (Ousterhout): rewards padding the implementation. We use depth-as-leverage instead.
- **"Interface" as the C# `interface` keyword or a class's public methods**: too narrow — interface here includes every fact a caller must know (invariants, server-authority rules, Command/Rpc ordering, SyncVar batching timing, required config).
