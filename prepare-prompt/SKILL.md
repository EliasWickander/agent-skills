---
name: prepare-prompt
description: Prepare a prompt that hands this session's work to a fresh agent session — to continue it, check it, or pick up what was left open. Use when asked for a prompt, brief, handoff, or continuation prompt for the next session or a subagent.
---

# Prepare prompt

The reader holds only this text — no chat history. Include everything it needs and nothing else.

## Workflow

1. **Settle the job.** Fix the reader (a fresh interactive session by default; a subagent only
   when asked) and the job type: *continue* the work, or *check* it (verify, review). Draft at
   once; ask only what this session doesn't settle, one question at a time, with a recommended
   answer.

2. **Collect the content.** Two lines are a complete prompt if they cover the case.
   1. **Situation** — what exists and what is open, each with its identifier (branch, commit, PR).
   2. **Task and deliverable** — what to do, the form of the result (chat report with a given
      structure, file, code change), and for a continue job, the check that says it is done.
   3. **Boundaries** — what it may decide alone and what it must not touch (read-only, no git
      writes). This and the deliverable form are the parts most often missed.
   4. **Facts it cannot derive** — paths, docs to read, tool quirks with their workaround,
      time-saving commands, approaches already ruled out.

3. **Cut.**
   - One line per fact, never how it was reached. Drop narration and anything it will redo anyway.
   - **Continue job:** keep the decisions it builds on and each ruled-out approach with its reason
     (`X, not Y: reason`).
   - **Check job:** drop this session's conclusions about the thing checked and all ruled-out
     approaches — they would steer the check.
   - Point, never paste: reference files and commits by path, section or hash. Write out only what
     lives nowhere else, such as a decision made in chat.
   - Never include a credential. Name where it lives (env var, config path), or tell the reader to
     get it from the user.
   - If an installed skill covers the task, open with its slash command and pass the rest as
     arguments; prose like "run the skill" cannot load a skill that blocks model invocation. A
     subagent cannot run slash commands: name the skill for it to invoke instead.

4. **Write it.** Imperative prose, no headings, no conversational voice, no nested fence; backticks
   for commands and paths except the opening slash command. Drop empty parts:

   ````text
   ```
   /<skill> <arguments>

   <Situation.>

   <Task, deliverable, done check.>

   <Boundaries.>

   - <Fact it cannot derive.>
   ```
   ---
   <Optional note to the user.>
   ````

5. **Check, then print.** All must hold:
   - Every path, id, command and skill was verified by a read-only lookup in this session.
   - Every settled fact was verified or stated by the user; inferences are marked "unverified" or
     cut.
   - The cuts of step 3 hold for the job type.
   - The reader can act without asking what was meant.

   If something can't be verified, cut it or mark it unverified; if the prompt can't work without
   it, stop and ask the user. Every revision reprints the whole block.

## Boundaries

- The printed prompt is the product: write no file unless asked.
