---
name: code-cleanup
description: Audit code for convention violations, duplication, dead code, hygiene issues, and local refactoring opportunities, then apply approved behavior-preserving fixes. Use when the user wants to clean up code, enforce project conventions, remove dead code, consolidate duplication, or tidy a folder or the current diff before committing.
---

# Cleanup

List findings that cross into architecture under **Escalate to architecture** and point the user to `/improve-codebase-architecture`. Read [REFACTORING.md](REFACTORING.md) when evaluating local refactors or deciding whether a finding crosses that boundary.

## Sources of truth

- `AGENTS.md` — conventions and excluded generated/third-party paths. Check against them; don't restate them in findings beyond naming the rule.
- `.agents/skills/game-design/SKILL.md` — intended gameplay, for checking data-driven code violations.
- `.editorconfig` — the compiler-enforced subset of the conventions. When it disagrees with `AGENTS.md`, `AGENTS.md` wins.

## Scope

Use the scope the user names, or infer it from the request:

- **Diff** — files changed against `HEAD` plus untracked files (`git --no-pager diff --name-only HEAD`, `git --no-optional-locks status --porcelain`). Default when the user is about to commit.
- **Path** — a folder or file the user names.
- **Codebase** — all of `Assets/Scripts/`. When dispatching sub-agents to parallelize the sweep by folder or category, instruct each to read `AGENTS.md`, [CHECKLIST.md](CHECKLIST.md), [REFACTORING.md](REFACTORING.md), and [SAFETY.md](SAFETY.md), restrict its scope to its assigned paths/categories, and return findings formatted using the Step 2 schema. Include that schema and [Risk tiers](#risk-tiers) in the hand-off, since sub-agents don't read this file.

## Risk tiers

Every finding gets exactly one tier:

- **Mechanical** — behavior-preserving by inspection and local: formatting, member order, explicit `private`, unused `using`, braces, attribute placement. Safe to batch.
- **Judgment** — behavior-preserving by argument: extract method, deduplication, dead-code removal, hot-path fixes. Each needs review.
- **Serialized** — touches anything Unity or Unity-Weld resolves by name or GUID: serialized fields, `[Binding]` members, methods wired through UnityEvents, `MonoBehaviour`/`ScriptableObject` class or file renames, assets. When reporting or applying a `Serialized` finding, read [SAFETY.md](SAFETY.md) and include the required migration steps.

A finding that also fixes a real bug — a missing unsubscribe, an unvalidated Command, a static that leaks between play sessions — is additionally labelled **Behavior change**. Call these out; never hide them inside a cleanup batch.

## Process

### 1. Sweep

Read [CHECKLIST.md](CHECKLIST.md) and work through every category for the chosen scope. Grep for mechanical patterns; read files for judgment calls. Before declaring any C# symbol or script unused or proposing any rename, run the reference checks in [SAFETY.md](SAFETY.md).

Group repeated mechanical violations into one finding with a file count and a few examples.

### 2. Present findings in chat

Return a concise Markdown report grouped by category. Skip empty categories.

For each finding:

- **Files** — paths, with line numbers where useful
- **Issue** — what's wrong and which rule or principle it breaks
- **Fix** — the concrete change
- **Tier** — `Mechanical`, `Judgment`, or `Serialized`, plus `Behavior change` if applicable
- **Why it's safe** — one-line behavior-preservation argument (omit for Mechanical)

After the findings:

- **Escalate to architecture** — findings beyond cleanup scope, one line each.
- **Promote to tooling** — recurring mechanical violations `.editorconfig` could enforce, and any `.editorconfig` / `AGENTS.md` conflicts.
- **Suggested order** — which batches to apply first. Usually Mechanical → Judgment → Serialized, with Behavior change findings as their own batch.

Do not edit anything yet. End by asking the user: "Which of these should I apply?"

### 3. Apply approved fixes

Only after the user approves:

- Apply one batch at a time so each can become its own commit. Never mix Mechanical and Judgment changes in one batch.
- Re-run the [SAFETY.md](SAFETY.md) reference checks immediately before any deletion or rename.
- After each batch, check diagnostics on touched files, then summarize what changed and what to play-test (see "Verification" in [SAFETY.md](SAFETY.md)).
- If a fix turns out to need more than the finding described, stop and report instead of expanding scope.
- Do not commit unless asked. If asked, one commit per batch.
