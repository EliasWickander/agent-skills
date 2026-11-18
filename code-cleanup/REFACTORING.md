# Local Refactoring

## In scope

- **Guard clauses** — flatten nested conditionals with early returns.
- **Extract method** — a long method doing several steps, or a block that needs a comment to explain it. Name the new method by intent; the comment usually becomes the name.
- **Extract pure logic** — calculations tangled with Unity or Mirror calls move to a plain C# static function in `Math/`, so they can be checked without a live session.
- **Name a condition** — replace a complex boolean expression with a well-named local or a question-phrased method (`IsInRange()`).
- **Inline** — private methods or locals called once that add no naming value.
- **Remove dead parameters** — parameters never read, or always passed the same value. Only when every caller is in this codebase and the method isn't invoked by Unity, Unity-Weld, UnityEvents, or Mirror.
- **Consolidate conditional fragments** — identical code at the start or end of every branch moves outside the branch.
- **Remove dead branches** — conditions that are always true or false given the code's own invariants.
- **Introduce named constant or SO field**
- **Rename for clarity** — names that no longer describe what the code does. Serialized tier if the member is serialized, bound, or a Unity/Mirror entry point.
- **Coroutine → Awaitable** — preserving cancellation on destroy.
- **Split a component** — a `MonoBehaviour` doing two unrelated jobs, where both halves stay on the same GameObject and no other module's interface changes. Always Serialized tier: prefabs need the new component and field values migrated.

## Escalate when

Stop and list the finding under **Escalate to architecture** if the refactor would:

- Move a responsibility between systems (e.g. from `Game/Units/` to `Game/Combat/`).
- Introduce or remove an interface, seam, adapter, or SO event.
- Change the networked surface (Commands, Rpcs, SyncVars) beyond a rename.
- Change a module's public interface, except through a rename or a refactor in the "In scope" list.
- Fix duplication whose real cause is a missing module that should own the concept.
- Be easier to justify in terms of depth, seams, or locality than in terms of readability.

## Behavior-preservation argument

Every Judgment refactor states why behavior is unchanged: same outputs for the same inputs, same side effects in the same order, and same timing relative to Unity and Mirror callbacks.

Unity and Mirror pitfalls that break "obviously safe" refactors:

- **SyncVar assignment order** — hooks fire synchronously on the server at assignment. Reordering assignments, or extracting them into a helper that runs in a different order, changes what hooks observe.
- **Lifecycle placement** — moving code between `Awake`, `OnEnable`, `Start`, and Mirror callbacks changes ordering relative to other objects and to SyncVar delivery.
- **Subscription timing** — moving a subscription out of `OnEnable` changes behavior when the object is disabled and re-enabled.
- **Server guards** — a helper extracted from a `[Server]` method must keep the guard or be private and only reachable from guarded paths.
- **Short-circuit order** — reordering conditions changes which side effects run and whether a destroyed-object check happens first.
- **Merging near-duplicates** — check that the copies don't differ in authority (one server-side, one client-side), in null handling, or in edge cases before merging.
- **Allocation** — extracting into a lambda or LINQ chain can add allocations on a hot path.
