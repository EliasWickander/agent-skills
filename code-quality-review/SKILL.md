---
name: code-quality-review
description: "Reviews one change (a branch, uncommitted work, or a commit range) against an extremely strict bar for structure and maintainability: abstraction quality, giant files, and spaghetti-condition growth, ending in an approve or changes-requested verdict. Use when the user asks for a thermo-nuclear code quality review, thermonuclear review, deep code quality audit, or especially harsh maintainability review."
disable-model-invocation: true
---

# Code Quality Review

Read-only: report findings, never edit files. Be ambitious: look for restructurings that keep behavior and make the change dramatically simpler, deleting whole branches, helpers, modes, or layers instead of rearranging them. Assume such a move often exists, frequently one that uses the existing architecture more effectively. Use the terms in the [improve-codebase-architecture glossary](../improve-codebase-architecture/SKILL.md#glossary).

## Workflow

1. **Resolve the scope**, then drop the generated and third-party paths `AGENTS.md` excludes:
   - The user names a commit range → that range.
   - On a branch other than `main` → `git --no-pager diff main...HEAD`.
   - On `main` → uncommitted changes: `git --no-pager diff HEAD` plus untracked files from `git --no-optional-locks status --porcelain`.

2. **Read the change in context.** Read every changed file in full, not only the hunks, plus the modules whose interface the change touches. Load [game-design](../game-design/SKILL.md) when the change touches gameplay. Compare each changed file's line count at the base with its count now.

3. **Review against the [Standards](#standards).** For each suspected problem, apply the deletion test and ask whether a different structure would remove it rather than move it.
   - Skip convention violations and local tidy-ups (naming, member order, braces, dead code, a single extract-method); they belong to `/code-cleanup`.
   - If the fix reaches into systems the change does not touch, list it under **Outside this change** instead of designing it.

4. **State the serialized-data cost** of every proposed restructuring: which prefabs, scenes, `.asset` files, or Unity-Weld bindings it affects and what migration they need, per the Serialized data area in [impact areas](../shared/impact-areas.md) and [Renaming safely](../code-cleanup/SAFETY.md#renaming-safely). Write `None` when nothing in that area applies and it touches no serialized field, `MonoBehaviour` or `ScriptableObject` class, `[Binding]` member, or asset.

5. **Decide the verdict.** A finding is a **blocker** when it breaks Standards 1–6 and nothing in the change (code comments, commit messages, or the user's explanation) justifies it. Any blocker → `Changes requested`; otherwise `Approve`. Working behavior alone never earns `Approve`.

6. **Report** with the [template](#report-template), findings in [priority](#priority) order. Prefer a few high-conviction findings over a long list; leave out nits when structural issues exist. If you skipped convention issues in step 3, end with one line suggesting `/code-cleanup`.

## Standards

1. **Delete complexity, don't move it.** When a plausible reframing would make branches, helpers, modes, or layers disappear, push for it. A refactor that moves code without reducing the concepts a reader must hold is not an improvement. Try these first: reframe the state model so conditionals disappear rather than move to one place; make the feature a natural extension of an existing abstraction; turn special cases into a default flow with fewer exceptions; separate orchestration from decision logic.
2. **No spaghetti growth.** New special cases, one-off booleans, nullable modes, flags, or "temporary" branches inserted into existing flows are design problems, as is a cohesive module becoming more coupled, more stateful, or harder to scan. Move the logic behind its own module, state model, or dispatcher.
3. **File size.** A change must not push a file from under 1,000 lines to over 1,000, unless a structural reason holds and the file stays clearly organized.
4. **Logic in the owning system.** Feature-specific logic stays out of shared systems, implementation details stay behind the interface, existing helpers are reused instead of near-duplicated, and repeated logic is extracted instead of copy-pasted.
5. **No shallow modules.** Pass-through helpers, identity wrappers, and abstractions that fail the deletion test add indirection without leverage.
6. **Direct over magic.** Generic mechanisms that hide a simple data shape, casts, `object`-typed payloads, nullable parameters, or silent fallbacks over an unclear invariant: make the type or the seam explicit instead.
7. **Atomic, simple orchestration.** Flag, when the cleaner structure is obvious: related state updates that can be observed half-applied, and independent work run in sequence for no reason where running it in parallel would also simplify the flow. Never a blocker on its own.

Be direct: do not soften a major maintainability problem into a mild suggestion. If the change makes the codebase messier, say so.

## Priority

1. Structural regressions
2. Missed simplifications that would delete complexity
3. Spaghetti growth
4. Interface, type, and seam problems
5. File size
6. Legibility

## Report Template

```markdown
# Code Quality Review: <scope, e.g. `feature-branch` vs `main`>

**Verdict:** Approve | Changes requested — <one-line reason>

## Findings
### 1. <title> (Blocker | Non-blocking)
- **Files:** <paths with line numbers>
- **Problem:** <what got worse, and which standard it breaks>
- **Proposed restructuring:** <the simpler structure; a short before/after when it helps>
- **Serialized-data cost:** <affected assets and migration, or `None`>

## Outside this change
- <problem in a system the change does not touch> → `/improve-codebase-architecture`; or `None`
```

## Stop Conditions

- If the scope is empty after excluding generated and third-party paths, say so and stop.
- If `main` or the named commit range does not resolve, stop and ask the user for the base.
