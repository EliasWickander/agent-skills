---
name: plan-in-steps
description: Plan an implementation as a sequence of isolated, individually reviewable steps, then execute one step at a time and stop for review between steps. Use when the user asks for an implementation plan, a step-by-step plan, or a breakdown of a feature or bug fix, and when they say to start, continue, or do the next step or step N of an existing plan.
---

# Plan in Steps

Determine which phase applies:

- No plan exists yet → **Planning**.
- A plan file exists and the user says "start", "next", "continue", "do step N", or gives review feedback → **Execution**.

## Plan file

- Location: `.plans/<short-kebab-slug>.md` at the repository root.
- Before writing, if `.plans/` isn't already ignored, add it to `.git/info/exclude`, never to `.gitignore`.
- If the user refers to "the plan" and several plan files exist, ask which one.
- Keep the plan lean: rules live in this skill, not in the plan.

Format:

```markdown
# Plan: <title>

Goal: <one or two sentences: what is true when the plan is complete>
Spec: <optional; `Docs/Specs/<feature>.md` when one covers this change>

## Step 1: <imperative title>
- Status: todo | review | done
- Change: <what this step changes, concretely>
- Files: <optional; only when known>

## Step 2: ...

## Step N: Validate
- Status: todo
- Change: <the project's automated checks, and manual checks for the user>

## Notes
<decisions, open questions, plan amendments with a short reason>
```

## Planning

1. **Investigate first.** Read the relevant code and data before proposing steps. Don't plan against guessed file paths or structures. If the conversation already contains an investigation of this change, build on it and read only what it does not cover. Ask the user only for decisions you cannot resolve from the codebase.
   - If a spec under `Docs/Specs/` covers this change, read it. Its Implementation Decisions, Boundaries, and Out of Scope constrain the plan. If its status is `Draft`, list its Blocking questions and stop until the user answers them. Use the Non-blocking defaults unless the user overrides them.
2. **Decompose by kind of change, not by feature.** Each step contains exactly one kind of change:
   - Refactor: no behavior change (e.g., move, split, slice, rename, extract, reformat).
   - Behavior: the actual fix or feature.
   - Editor: work the user does by hand in a tool the agent cannot drive. `Change` lists the exact actions.
   - Cleanup: removing dead code or temporary scaffolding after the change lands.
   - Docs: `Docs/Specs/`, the `game-design` skill, other documentation, or comment-only changes.
3. **Order steps** in this default sequence, deviating only with a stated reason:
   1. Refactor steps that make the change easy.
   2. Behavior steps, smallest coherent units first, each followed by the Editor step that wires it, if any.
   3. Cleanup.
   4. Docs.
   5. **Validate**, always the final step.
4. **Write `Change` precisely** enough that the user would notice if the diff did something else. If the change could reasonably be read as applying more broadly than intended, state the boundary (e.g. "only in the new slice; leave the old slice untouched").
5. **Write the Validate step's `Change`** as the project's automated checks plus the manual checks the user must do themselves.
   - With a spec, reference its acceptance criteria and end-to-end walkthrough by number instead of copying them (e.g. "Spec AC 1–14, end-to-end walkthrough"), and add only checks the spec does not cover.
6. **Self-check before presenting.** Revise the plan until all of these hold:
   - No step mixes two kinds of change. If it does, split it.
   - Each step's diff is understandable without reading another step's diff.
   - Each step leaves the project compiling.
   - No over-splitting: adjacent tiny steps of the same kind touching the same area are merged.
   - The last step is Validate.
   - With a spec: Validate covers every acceptance criterion, and no step contradicts its Boundaries or Out of Scope.
7. **Write the plan file**, then present a compact summary (numbered step titles) and the file path. Tell the user they can edit the file directly. **Stop and wait for approval.** Do not start step 1 in the same turn.

## Execution

Each turn executes **at most one step**.

1. Re-read the plan file. Pick the step the user named, or the first step with `Status: todo`. If a previous step is still `review`, the user's message approves it: set it to `done` unless they gave change requests (see step 7).
2. Load any available skill that matches this step's work.
3. Implement **only** that step's `Change`. Anything covered by another step, or by no step, is out of scope.
4. **Scope discipline:** if you discover that something outside this step must change (a missing refactor, a bug elsewhere), do not make that change. Finish what is possible within scope, then propose a plan amendment (a new step, a reordering, or an edit to a later step) and record it under `Notes` as *proposed*.
5. **Sanity check** the step with the cheapest relevant check: `diagnostics` on the touched files, or the project's build command. Leave work the user must do by hand for an Editor step. Do not run manual checks here; that is the Validate step's job. If the check fails because of this step, fix it within scope.
   - For an Editor step, skip implementation and the sanity check: give the user the step's actions as a checklist, then continue at the next item.
6. Set the step's `Status: review`, then **stop** and report:
   - Step number and title.
   - Files changed, with a one-line description each.
   - Sanity check run and its result (never claim a pass you didn't see).
   - Proposed plan amendments, if any.
   - The next step's title, so the user knows what "continue" will do.
7. **Review feedback** applies to the current step only. Make the requested changes, re-run the sanity check, keep `Status: review`, and stop again.
8. Do not commit unless the user asks. If they want a commit per step, use the step title as the subject line.

### Validate step

1. Run the project's automated checks.
2. Report results, and list the manual checks the user still needs to do. With a spec, list each acceptance criterion by number so the user can tick them off.
3. If anything fails, do not fix it in this step. Propose a new fix step inserted before Validate, record it under `Notes`, and stop. After the fix step is reviewed, run Validate again.

When Validate is `done`, give a short summary of the full change and ask whether to delete the plan file.
