---
name: compare-designs
description: Compare several valid solutions or architectures for a design decision against explicit criteria, grounded in the codebase, and recommend one with the evidence that would change the recommendation. Use when asked to compare approaches, weigh tradeoffs, or choose between designs or architectures ("X vs Y", "which approach should we use").
argument-hint: 'decision to make, or "the investigation above"'
user-invocable: true
---

# Compare Designs

This is a read-only workflow. Do not modify files. Recommend a design; do not plan its implementation.

## Workflow

1. **State the decision** as a single choice, such as "Where should X's logic live?".
	- If the conversation already contains an investigation, reuse its brief, affected systems, and existing patterns; read only what it does not cover.
	- Otherwise collect a minimal brief (problem, goal, hard constraints, and the most concrete code anchor) and read the owning code before proposing options.

2. **Classify reversibility** with a one-line reason.
	- **Hard to reverse:** changes that saved data, authored content, or other code builds on, how state is split between server and clients, or anything the user has to redo by hand in a tool. Run the full comparison.
	- **Easy to reverse:** internal to one system, or replaceable without touching anything hard to reverse. Run a light comparison: up to three options and no sub-agents.

3. **Set the criteria before generating options.** Start from the [impact areas](../shared/impact-areas.md), plus consistency with current codebase patterns, implementation cost, and maintenance cost. Drop those that do not apply and add any the brief implies. Weight each (high, medium, low) with a reason tied to the brief, and mark hard constraints as **must-pass**. If a new criterion surfaces later, add it, say why, and re-score every option.

4. **Generate distinct options**, up to five.
	- Include the option that follows the current codebase pattern, if one exists; if that pattern is outdated or being migrated away from, say so with evidence. Include the minimal option that meets the must-pass criteria.
	- Options must differ in structure (where logic lives, who owns the data, how control flows, which extension mechanism is used); merge options that differ only in detail.
	- Include only options you would recommend under some plausible weighting of the criteria, with evidence they could work in this codebase. Do not add an option just to reach five or to make the recommended option look better by comparison.

5. **Ground each option** by filling in its entry in the report's Options section, with file references.
	- Load any available skill that owns a touched area and follow its rules.
	- Ground independent options in parallel with sub-agents. Give each the brief, the criteria, and every option's summary; ask it to make the strongest evidence-backed case for its assigned option and to list where that option is weaker than the others. Judge their reports yourself.

6. **Score and eliminate.** Score every option against every criterion as `++`, `+`, `0`, `-`, or `--` with a few words of justification, or `?` where evidence is missing. Eliminate options that fail a must-pass criterion or are dominated by another option. Use the weights to explain the judgment, not to compute a numeric total.
	- If parts of two surviving options combine into an option likely to score better than either, add it as a hybrid: ground it as in step 5, score it, and eliminate again.

7. **Recommend** one option.
	- State the flip conditions: the evidence that would make a named alternative better.
	- For the biggest `?` or riskiest assumption, propose the cheapest spike that resolves it and which result points to which option.
	- If two options remain genuinely tied and no hybrid resolves the tie, record the choice as a blocking decision instead of forcing a pick.

8. **Report** in chat using the template below, then:
	- If another skill's workflow invoked this one and there are no blocking decisions, return the recommendation to that workflow.
	- Otherwise stop. Offer [grill-me](../grill-me/SKILL.md) to stress-test the decisions and flip conditions, and [plan-in-steps](../plan-in-steps/SKILL.md) for the recommended option, recording the chosen option, rejected options, flip conditions, and decisions under the plan's `Notes`.

## Report Template

```markdown
# Design Comparison: <decision>

**Reversibility:** Hard | Easy — <reason>

## Criteria
| Criterion | Weight | Must-pass | Why it matters here |
|-----------|--------|-----------|---------------------|

## Options
### A. <name> (<follows current pattern | minimal | alternative | hybrid of <options>>)
<Approach in two or three sentences>
- **Changes:** <systems and files with links, extension points used, and anything new it requires>
- **Precedent:** <existing code, history, or PR; or `None`>
- **Failure modes:** <what goes wrong, or `None known`>

## Comparison
| Criterion | A | B | C |
|-----------|---|---|---|

## Eliminated
- <Option>: <failed must-pass criterion, or dominated by which option>; or `None`

## Recommendation
**<Option>**: <deciding criteria and the tradeoff accepted>

- **Would flip to <option> if:** <specific evidence>
- **Suggested spike:** <cheapest experiment and how to read its result, or `None needed`>

## Decisions
1. **<decision>** (blocking | non-blocking): <options> → Recommended: <answer and reason>
```

## Stop Conditions

- If the decision cannot be stated as a single choice, ask the user to narrow it.
- If the problem is not understood well enough to name the affected systems, stop and recommend investigating it first.
- If only one credible option exists, say so with a one-line reason for each discarded idea, and skip the full template.
- If no option passes the must-pass criteria, report which constraint blocks each option and stop. Do not relax constraints silently.
- If the comparison shows the decision is framed wrong (the real choice is elsewhere, or a prerequisite decision is open), report that and ask before continuing.
