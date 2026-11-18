---
name: investigate-feature
description: Investigate a new or substantially extended feature from a spec, design doc, or description; map the affected systems, choose a design direction, surface decisions and risks, then offer to turn the findings into a spec with to-spec or a step-by-step plan with plan-in-steps. Use when asked to investigate, scope, spike, research, or plan a larger feature that spans multiple files or systems.
argument-hint: 'Docs/Specs or Docs/Design path, or feature description'
user-invocable: true
---

# Investigate a Feature

Stay read-only until step 9.

## Workflow

1. **Resolve the feature source.** Accept a spec path under `Docs/Specs/`, a design doc path under `Docs/Design/`, or a written description.
	- For a spec or design doc, read it and any specs, design docs, or sources it references.
	- For a design doc, also read the feature's spec in `Docs/Specs/` if one exists. The design doc's rules are decided: do not reopen them as Decisions. If one conflicts with the code or an invariant, handle it under Stop Conditions.
	- For a description, check `Docs/Specs/` and `Docs/Design/` for an existing spec or design doc. If several are plausible, show them and ask the user which one applies. If none exists, continue from the description.

2. **Write the feature brief.** State the problem and who has it (each kind of user, including people who author content or config), the goal, verifiable acceptance criteria, and non-goals, using what the source states. You may add acceptance criteria that the goal implies but the source does not state; mark each as **Proposed**. Where the source is silent on a choice between options that affects scope, add it to the report's Decisions with your recommended answer instead.

3. **Record the starting point.** Note the current branch and commit, and whether the worktree has uncommitted changes. Do not switch branches or alter the worktree. If the feature targets a different branch, state the mismatch.

4. **Map the affected systems.** For each subsystem the feature touches, find the owning code and data, its entry points, data flow, and existing extension points. Start from the most concrete anchor in the brief: a component, command, screen, API, data file, or symbol.
	- When the subsystems are independent, investigate them in parallel with sub-agents, giving each the brief and one subsystem, and asking for owning files, extension points, and constraints with file references.
	- Load [game-design](../game-design/SKILL.md) and any other available skill that owns a touched area, and do not restate project conventions in the report.

5. **Find existing patterns.** Search the codebase, git history and `Docs/Specs/` for similar features and earlier attempts. Before treating a pattern as the one to follow, check that it is current: look for a newer replacement, an in-progress migration away from it, deprecation or legacy markers, or known bugs and hard-coding it causes. Prefer extending a current pattern over inventing a new one. If the closest pattern is outdated or flawed, say so with evidence and carry both following it and the better approach into step 6.

6. **Choose a design direction.** Identify the option that follows the current pattern and any credible alternatives. If one option clearly wins, recommend it, give a one-line reason for each rejected option, and state what evidence would change the recommendation.
	- Escalate to [compare-designs](../compare-designs/SKILL.md) instead when two or more options remain credible, the choice is hard to reverse (as defined in compare-designs), or the closest pattern is outdated. Pass it the brief, affected systems, and existing patterns, then continue at step 7 with the option it recommends.

7. **Assess impact.** For the recommended design, check each of the [impact areas](../shared/impact-areas.md) and record findings, or `None` with a one-line reason. Under Verification, name the check that covers each acceptance criterion.

8. **Report the investigation** in chat using the template below, then **stop** and offer two next steps:
	- Create an implementation plan from it (step 9).
	- Write a spec first: the user runs `/to-spec` in this thread, then asks for the plan. Recommend this when another agent or thread will implement the feature, or when it needs lasting Given/When/Then acceptance criteria. `to-spec` is user-invoked only; do not run it yourself.
	- Blocking decisions must be answered before planning. Non-blocking decisions use the recommended answer unless the user says otherwise.
	- Offer [grill-me](../grill-me/SKILL.md) if the user wants to stress-test the decisions.

9. **Plan on confirmation.** Only when the user confirms, load [plan-in-steps](../plan-in-steps/SKILL.md) and run its Planning phase for the recommended design, using the feature's spec if one exists, including one written after the report. Make the plan's Validate step check every acceptance criterion, including Proposed criteria and criteria added by decisions. Record the chosen design option, the rejected options with a one-line reason each, the conditions that would change the choice, the decisions with their answers, the Proposed criteria, and the open risks under the plan's `Notes`. If the feature is too large to review as one plan, propose splitting it into separately planned parts and ask which part to plan first.

## Report Template

```markdown
# Feature Investigation: <title>

**Source:** <spec or design doc path, or "User description">
**Investigated checkout:** <branch and commit; uncommitted changes or branch mismatch, if any>

## Brief
- **Problem:** <what is missing or painful today, and for whom>
- **Goal:** <what is true when the feature ships>
- **Acceptance criteria:** <from the source; mark criteria you added as **Proposed**>
- **Non-goals:** <from the source, or `None stated`>

## Affected Systems
| System | Owning code / data | Extension point | Notes |
|--------|--------------------|-----------------|-------|

## Existing Patterns
- <Similar feature or earlier attempt, whether it is current or outdated, with links; or `None found`>

## Recommended Design
<Approach, changed systems, and why it fits existing patterns>

**Alternatives considered:** <Only real alternatives, with the reason each was not chosen; or `None`. If compare-designs ran, summarize its recommendation instead of repeating the comparison.>

**Would change if:** <evidence that would change the recommendation>

## Impact
| Area | Finding |
|------|---------|
| <one row per impact area> | |

## Decisions
1. **<decision>** (blocking | non-blocking): <options> → Recommended: <answer and reason>

## Risks and Open Questions
- <Remaining uncertainty, or `None`>
```

## Stop Conditions

- If the spec or a linked document cannot be accessed, stop and ask the user for access or its contents. Do not reconstruct requirements from partial search results.
- If the brief's core goal remains ambiguous after reading the source, ask before mapping systems.
- If investigation shows the feature conflicts with an existing system or with an invariant in `game-design` or `AGENTS.md`, report the conflict and stop before planning a design that works around it.
- If the scope grows beyond what the brief describes, record the extra scope under **Risks and Open Questions** and ask before investigating it.
