---
name: self-improve
description: Turn a user's correction into a durable fix in the skill, doc, or check that should have steered the agent, or propose a new one when none fits. Use when the user rejects, reverts, or overrides the agent's output or approach on something a skill, doc, or check covers or should cover, or states a standing preference ("we always…", "never…").
---

# Self-Improve

"Skill/doc" means any standing instruction (`SKILL.md`, `AGENTS.md`, convention doc, rules file); a "check" is a lint rule, hook, or CI job.

## Workflow

1. **Capture the lesson.** Finish the user's current task first, then offer to persist the lesson. State in one line the general rule the feedback implies, not the incident ("Mock external HTTP in unit tests", not "the agent mocked the wrong call"). Drop one-off, task-specific tweaks. Move on when each lesson is one durable rule.

2. **Locate the target.**
	- **Scope:** would the lesson hold in another project? No → a project doc, rule, or wrapper, never a shared generic skill. Yes → the governing skill or doc, edited at its canonical source when the installed copy is a symlink or sync.
	- **Find the owner:** search skills (including `shared/`), `AGENTS.md`, and the docs it points to. If the repo has a doc-placement guide, follow it.
	- **Rule already exists but did not fire:** find why (buried, in a doc the agent would not open for this action, scoped or worded too narrowly, unenforced) and fix that cause.
	- **Mechanical lesson** (any fixed pattern: banned API, import shape, file location) → propose a check. Read the repo's analyzer and lint config, check scripts, and CI first; extend or fix an existing check before adding one. Write a prose rule only if the user declines the check or no tooling can host it.
	- **No fitting target** → propose a new skill (recurring workflow), rule (standing constraint), or the closest doc. A new doc needs an index line and a "read when" hook in `AGENTS.md`.
	- **Context cost:** always-loaded docs hold mostly pointers; a review-only rule goes in a review-time doc.
	- **Several lessons:** propose all placements in one table first.

	Move on when every lesson has a target the user accepted.

3. **Draft the edit.** Write the least text that fully captures the rule: agent-agnostic ("the agent", never a vendor name), no process narration. Prefer tightening an existing rule over adding one. Lessons sharing a target become one diff. Draft a check outside the repo and run it on a reproduction of the mistake (must fail) and on the corrected case (must pass). Move on when each proposal is filled in per the template below.

4. **Apply on approval.** Done when every lesson is applied, declined, already covered, or waiting on the user.

5. **Report** with the closing template below.

## Report Template

Proposal, one per target:

````markdown
### <one-line lesson>

**Target:** `<path>` — <existing | new, indexed in `AGENTS.md`> <; canonical source `<path>` if symlinked or synced>
**Why the rule did not fire:** <only when the rule already existed>
**Reverses:** <old rule> → <feedback> → <replacement> <only when it reverses a rule>
**Status:** not applied — awaiting approval

```diff
<change>
```

**Check proof:** fails on mistake: <result>; passes on fix: <result> <or: untested — reason> <only for a check>
````

Placement table, when there are several lessons:

```markdown
| Lesson | Target | Existing / new |
| ------ | ------ | -------------- |
```

Closing report:

```markdown
| Lesson | Result | File |
| ------ | ------ | ---- |
| <lesson> | applied / declined / already covered / awaiting user | `<path>` |
```

## Stop Conditions

- Never edit a skill, doc, or check without approval of the concrete diff.
- If a lesson reverses an existing rule, stop and show the old rule, the feedback, and the replacement; the feedback may be context-specific rather than a true reversal.
- If you are unsure whether a lesson is durable, where it belongs, or whether a new target is needed, ask before drafting.
- If the target is not writable or its canonical source cannot be found, stop on that lesson and ask; continue the others.
- If a check cannot be run here, label it untested; never claim it passes.
