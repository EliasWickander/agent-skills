---
name: improve-codebase-architecture
description: Find deepening opportunities in a codebase. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, or make a codebase more verifiable and AI-navigable.
---

# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. The aim is verifiability and AI-navigability.

No API or network-protocol backwards compatibility is needed.

## Glossary

Use these terms exactly in every suggestion. Don't drift into "component," "service," "API," or "boundary." When a term's boundary is ambiguous (e.g. distinguishing an **adapter** from an **implementation**, or internal vs. external **seams**), read [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config, performance.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place.
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge, and verification concentrated in one place.

Key principles:

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the verification surface.** If you need to verify past it, the module is probably the wrong shape.
- **One adapter = hypothetical seam. Two adapters = real seam.** Don't introduce a seam unless something actually varies across it.

## Process

### 1. Explore

Read [DEEPENING.md](DEEPENING.md) before evaluating candidates.

Explore the codebase without following rigid heuristics; note where you experience friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow**?
- Where has a manager or singleton accumulated unrelated responsibilities — a god module whose interface keeps growing because everything routes through it?
- Where does logic poll state in `Update()` instead of reacting through SyncVar hooks or SO events, hiding the real data flow?
- Where do tightly-coupled modules leak across their seams — e.g. one behaviour reaching through `GetComponent` chains into the internals of several others?
- Where does a module work for the single case today but fail at the volume the design doc implies — hardcoded for one unit / one wave / one lane where the doc calls for many?
- Which parts of the codebase are hard to verify through their current interface — i.e. you can't exercise the behaviour without poking at internals or standing up a full host/client session?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

Project constraints while exploring:

- Exclude generated and third-party folders listed in `AGENTS.md`.
- Read `.agents/skills/game-design/SKILL.md` when evaluating gameplay modules: scale for what the design doc describes (many unit types, wave matchups, growing economy), and never invent seams for anything under "Systems Not Yet Designed".

### 2. Present candidates in chat

Return a concise Markdown architecture review in chat.

For each candidate, use the same template:

- **Files** — which files/modules are involved
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and how verification would improve
- **Before / After** — short diagram or bullet comparison illustrating the shallowness and the deepening
- **Hot-path impact** — whether the module runs per-frame or per-unit, and what the refactor does to allocations and dispatch there (omit if not on a hot path)
- **Serialized-data cost** — affected prefabs, scenes, `.asset` files, or bindings and their migration, per the Serialized data area in [impact areas](../shared/impact-areas.md); `None` when untouched
- **Recommendation strength** — one of `Strong`, `Worth exploring`, `Speculative`

End the review with a **Top recommendation** section: which candidate you'd tackle first and why.

Do NOT propose interfaces yet. End by asking the user: "Which of these would you like to explore?"

### 3. Follow-up design conversation

Once the user picks a candidate, walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what verification survives.

When the user wants to design or compare concrete interface alternatives for the chosen candidate:

1. Run [compare-designs](../compare-designs/SKILL.md) with the interface choice as the decision and the candidate write-up as its brief. Add depth, locality, and seam placement to its criteria. When it generates options:
   - Try each angle: the smallest interface (1–3 entry points), a trivial default case for the most common caller, flexibility for the scale `game-design` describes, and ports & adapters when a dependency crosses a seam. Keep only the angles that give a credible option.
   - Add a line to each option's entry listing its entry points (methods, Commands, SyncVars).
2. When it returns a recommendation, sketch that interface only:
   - **Interface** — types, methods, invariants, ordering, and error modes; for a module on the server/client seam, its networked surface (Commands, SyncVars and hooks, Rpcs)
   - **Usage** — a short example of a caller using it
   - **Hidden** — what the implementation hides behind the seam
   - **Dependencies** — their category from [DEEPENING.md](DEEPENING.md) and the adapters at each seam

Do not edit project files during this conversation unless the user explicitly asks for implementation changes.
