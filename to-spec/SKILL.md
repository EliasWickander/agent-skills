---
name: to-spec
description: "Writes a feature spec file from the current conversation without interviewing the user, for another agent to implement. Use when the user asks to write or update an implementation spec for a feature."
disable-model-invocation: true
---

The spec's reader is another agent that has the repo but not this conversation. Write the smallest spec that would let that agent implement the feature safely and prove it works.

- Only create the document. Do not commit or start implementing.
- Do not interview the user. Synthesize what you already know, and handle anything undecided as described in Undecided Items.
- Do not propose tests, test seams, or test assemblies.

## Process

1. **Confirm there is a feature to spec.** If the conversation does not settle what the feature does (only a loose idea, or several competing directions with no choice made), do not write a file. Tell the user which decisions are missing and stop.

2. **Gather context.** If you haven't already, read [game-design](../game-design/SKILL.md) and use its vocabulary throughout the spec. If it links a design doc for this feature, read that too and treat it as part of `game-design` everywhere below. Explore the parts of `Assets/Scripts/` the feature touches.

3. **Check the design status.** If the feature touches anything under "Systems Not Yet Designed" or marked "design note — not implemented" in `game-design`, those parts are undecided items unless the feature's design doc settles them.

4. **Check the size.** If the feature has more than one independently playable slice and you expect, or reach while writing, about 20 acceptance criteria, spec only the P1 slice. List the remaining slices under Out of Scope as follow-up specs and mention the split in your report.

5. **Write the spec** to `Docs/Specs/<feature-name>.md` (kebab-case), using the template below. Start the file with a `# <Feature Name>` title, then a `Status:` line: `Draft` while any Blocking question remains, otherwise `Ready for review`. Never set `Approved`; only the user sets it.
   - If a spec for the same feature already exists, update it instead of creating a second file, and recompute its status.

6. **Self-check** the file against the checklist below and fix every failure before reporting.

7. **Report** the file path and status, then the Blocking questions in ranked order, then the Non-blocking questions with their proposed defaults. If there are more than three Blocking questions, say that the feature needs another design pass before it is specced.

## Undecided Items

Anything the conversation, `game-design`, or the codebase does not settle is undecided. An inference counts as undecided, however likely it seems. Never present an undecided item as decided. List each one under Open Questions as exactly one of:

- **Blocking (B1, B2, ...)**: no default is safe, because a wrong guess would change scope, break fairness or hidden information, or force rework of a large part of the spec. Give no default, and say what the question blocks. Rank by impact: scope > fairness/hidden information > player experience > technical details.
- **Non-blocking (N1, N2, ...)**: a wrong guess is cheap to correct later. Give your proposed default and the reason for it. The implementer uses the default until the user answers. Where the spec relies on the default, write it as stated behavior followed by the question ID, e.g. "Hover labels use the owner's display name (N2)."

## Writing Rules

- **Self-contained.** Do not refer to "the discussion" or "as agreed". State the decision itself. Define any term that is not in `game-design` or `AGENTS.md`.
- **Minimal.** Every sentence must either constrain the implementation or help check it. Do not restate the same behavior in Solution, User Stories, and Acceptance Criteria. Leave a section out when it has nothing to say. These sections are always present: User Stories, Acceptance Criteria, Out of Scope, Open Questions.
- **Only what the repo can't tell.** Do not describe existing code, file layout, or patterns the implementer can read for themselves. Record decisions, the reasons behind non-obvious ones, and anything in the code that is misleading.
- **Names, not code.** Name modules, classes, and folders when it helps. No line numbers or code snippets, except the decision-rich parts of a prototype snippet (state machine, `EventData` shape, SyncVar layout), marked as coming from a prototype.
- **Tunables.** Name each tunable value, attribute it to a ScriptableObject, and give its default. Use the defaults from `game-design` where they exist, unless the conversation changes them. Collect defaults that neither `game-design` nor the conversation sets into a single Non-blocking question.
- **Checkable claims.** In Solution, Acceptance Criteria, and Boundaries, every claim must be checkable: could two people agree on whether it holds? Words like smooth, fast, intuitive, robust, properly, or as expected usually mark a claim that isn't. Fix the claim by stating the evidence you would accept, not by swapping in a synonym. For example, "blends smoothly" becomes "blends over `BlendDuration` (default 0.6 s) with no single-frame jump". Names such as a "quick-buy" button are fine.

## Acceptance Criteria Rules

- **Given/When/Then.** Write each criterion as one sentence: *Given* [state], *when* [trigger], *then* [observable result].
- **Atomic.** One trigger and one outcome per criterion. Several observations of that outcome are fine (no defender spawns, gold stays the same, a rejection message appears). A second outcome goes in another criterion.
- **Concrete values.** Use real keys, button names, label text, numbers, and ScriptableObject defaults. Never write "the correct label" or "the right amount".
- **Observable.** Someone who has never read the code can check it by hand in Play Mode, using the screen, the Inspector, or the Console. If you can't write it that way, move it to Open Questions.
- **Setup tag.** Start each criterion with `[Host]` (host alone, debug game) or `[Host + Client]` (host plus a ParrelSync client).
- **Traceable.** Tag each criterion with the user stories it proves, e.g. `(US 3)`. Every user story needs at least one criterion.
- **Coverage.** Group criteria under three headings. If a group truly has nothing, write "None" followed by the reason.
  - **Happy path**
  - **Edge cases**: boundary values, empty or full states, phase transitions mid-action, 2v2 vs 4v4, late joiners, what teammates and opponents must not see
  - **Failure and rejection**: invalid input, Commands the server rejects (and the TargetRpc feedback the player sees), missing data, disconnects
- **End-to-end check.** Finish with one numbered walkthrough that plays the feature through a realistic match flow, so the pieces are checked together. If the criteria already cover the whole flow, write "None, covered by criteria 1–3" (with the actual numbers).

<acceptance-criteria-example>
Bad:
- The shop handles gold properly.
- Unaffordable cards are clearly disabled and placement fails gracefully.

Good:

**Happy path**
1. `[Host]` (US 1) Given the player has 3 gold and the Archer card costs 4, when the planning phase starts, then the Archer card is greyed out.
2. `[Host]` (US 1) Given the Archer card is greyed out, when the player clicks it, then no placement highlight appears.

**Failure and rejection**
3. `[Host + Client]` (US 2) Given the client has 4 gold and is placing an Archer, when the client clicks a cell that already holds a defender, then no Archer spawns, the client's gold stays at 4, and the client sees the "Cell occupied" message.
</acceptance-criteria-example>

## Self-Check

Re-read the spec against Undecided Items, Writing Rules, and Acceptance Criteria Rules, then confirm:

1. Every rejection listed under Commands and validation has a criterion under Failure and rejection.
2. Nothing contradicts `AGENTS.md` or `game-design`, except design changes listed under Design Impact.
3. An agent with only this file and the repo could implement the feature without asking about anything outside Open Questions.
4. Cut pass: delete any sentence that restates another, describes code the implementer can read, or doesn't change what gets built or checked.

<spec-template>

## Problem Statement

The problem from the player's (or designer's) perspective, in a few sentences.

## Solution

The solution from the player's (or designer's) perspective. Describe behavior, not implementation.

## User Stories

A numbered list with one story per distinct need:

1. (P1) As a <actor>, I want <feature>, so that <benefit>

P1 stories together form the smallest playable slice and are implemented first. P2 and P3 stories build on it and could be cut or deferred.

Actors: player, teammate, opponent, host, designer (who tunes ScriptableObjects). Add teammate and opponent stories when visibility matters (hidden gold, private shop, fogged lanes), and 2v2 vs 4v4 stories when the feature scales with player count.

## Acceptance Criteria

Follow the Acceptance Criteria Rules.

## Design Impact

- Which `game-design` sections the feature implements or changes. If it changes the design, list the edits the `game-design` skill needs.
- The effect on the economy-vs-defense tradeoff or on balance, if any.

## Implementation Decisions

Decisions that were made. Cover what applies:

- **Modules**: what is built or modified, and their responsibilities.
- **Server state**: what lives on the server and how it syncs. What is client-local and never synced.
- **Commands and validation**: each client request, what the server validates, and which rejections send a TargetRpc vs. are silently dropped.
- **Events**: new or reused `GameEvent<T>` assets and their `EventData` structs, plus who raises and who listens.
- **Data**: new or changed ScriptableObjects, their tunable fields, and defaults.
- **UI**: ViewModels, their bindable properties and actions, and which Canvas they belong on.
- **Lifecycle**: phase transitions, match start/end, and late-joiner/initial-state handling.
- **Spawning and pooling**: networked prefabs, `ObjectPool<T>` use, and how objects are removed.

## Boundaries

Feature-specific rules only; do not repeat `AGENTS.md`.

- **Always**: obligations specific to this feature (e.g., "route every camera change through `PlayerCameraController`").
- **Ask first**: changes the implementer must check with the user before making (e.g., touching a shared system like `GameplayState`, changing an input asset's existing maps, adding a package).
- **Never**: tempting shortcuts that would break the design (e.g., "never sync camera state"). Forbid behavior, never the general seams `AGENTS.md` requires.
- **Performance**: limits with numbers, only when the feature adds per-frame, per-unit, or network cost (e.g., "at most one raycast per frame").

## Out of Scope

Adjacent work an implementer would be tempted to do but must not, each with a short reason when it isn't obvious.

## Open Questions

Follow Undecided Items. Write "None" for an empty list.

- **Blocking**
- **Non-blocking**

## Gotchas

Traps the implementer can't see from the code: misleading existing code, patterns that look reusable but aren't, timing or ordering pitfalls, and decisions that look wrong but are intentional.

</spec-template>
