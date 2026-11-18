---
name: cleanup-agent-skill
description: Rewrite a verbose agent skill to the shortest form that still makes the agent do the same thing. Use when asked to clean up, trim, or shorten a skill or SKILL.md file.
---

# Clean Up an Agent Skill

## Workflow

1. **Read the skill.** Read the target `SKILL.md` and every supporting file it references. If the user did not name a skill, ask which one.

2. **Write down the contract.** List what the skill must keep doing:
   - the trigger terms in `description`
   - each workflow step and its order
   - the output format
   - each stop condition

3. **Cut line by line.** For each sentence, ask: would the agent act differently without it?
   - No: delete it.
   - Yes, but it takes more words than needed: shorten it.
   - Yes, and it repeats content another skill or instruction file owns: replace it with a link to that file.

4. **Check the contract.** Compare the result against the list from step 2. Restore anything that was lost. Do not apply a change to `name` or the contract; list it for the user instead.
   Then account for every cut as [Account for every cut](../../../.github/instructions/ai-skills.instructions.md#account-for-every-cut) requires.

5. **Summarize** in this format:

   ```markdown
   ## Cleanup: <skill name>

   <lines before> → <lines after> lines

   ### Cut
   - <what was removed or shortened> — <where it still lives, or why the agent acts the same>

   ### For your decision
   - <change that would alter the contract> — <what it would change>
   - <sentence kept because you can't tell> — <how cutting it might change what the agent does>
   ```

   Leave out a section if it is empty.

## Stop Conditions

- If the skill covers more than one workflow, report this and stop. Splitting it is a redesign, not a cleanup.
- If you cannot tell whether a sentence changes the agent's behavior, keep it and ask the user.
- If a file the skill links to does not exist, report the broken link and do not guess its contents.
