---
description: "Conventions for creating, editing, and reviewing Agent Skills. Use when creating, editing, or reviewing a skill in .agents/skills/"
applyTo: ".agents/skills/**"
---

# Agent Skill Conventions

## Format

### Structure

```
my-skill/
├── SKILL.md          # Required: frontmatter + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
```

### Name

- Max 64 characters; lowercase letters, numbers, and hyphens only
- No leading, trailing, or consecutive hyphens
- Matches the parent directory name
- Specific to the task (`write-agent-skills`), never generic (`helper`, `utils`); follow the naming pattern of the existing skills

### Description

- Non-empty; max 1,024 characters; no XML tags
- 1–3 sentences: first what the skill does, in third person, then `Use when...` with the specific triggers
- Add `Do not use for...` only when a similar task could be confused with it

### Supporting files

- Move content to `references/` when `SKILL.md` nears 500 lines, or when only some tasks need it; keep the essential instructions in `SKILL.md`
- Link files in `references/`, `scripts/`, and `assets/` directly from `SKILL.md`, never from other references, and state when to read, run, or use each one

## Principles

### Declare an exact scope

The frontmatter description is the discovery surface. Focus it on user intent, not implementation: put the main use case first and name the relevant tasks, tools, file types, or outputs that should load it.

### Own one workflow

A skill has one clearly defined responsibility. If a task requires multiple distinct workflows, split it across separate skills rather than folding them into one. Keep skill boundaries clean and never create delegation loops between skills.

### Sequence the workflow into steps

Write the workflow as explicitly numbered steps rather than an unordered list. Each step states one action and the condition that must hold before moving on. State the required inputs and expected outputs, and make each decision point and its condition explicit.

### Define the output format

When a skill reports findings, proposals, or summaries back to the user, give it a concrete Markdown template or table schema to fill in.

### Define each rule once

A rule lives in exactly one skill or instruction file. Search existing files before adding guidance; if another file could apply to the same decision, explicitly defer to the owning file rather than stating another version of the rule.

### Link to the source of truth

When a skill references commands, paths, options, versions, or conventions that can change, link to the file or authority that defines them rather than copying them inline. Link to the specific file, not an open-ended reference. Keep a short example only when stable and necessary to perform the workflow.

If a required external source or tool is unavailable, stop and ask the user for guidance. If it is explicitly optional, skip it and report that limitation.

### Keep instructions minimal

Write only what the agent needs to complete the task correctly, in short, direct instructions with consistent terms. Do not restate context or concepts the agent already knows, and do not add defensive instructions for edge cases that are not expected to occur. Give exact steps for fragile tasks and allow flexibility elsewhere. Explain non-obvious rules briefly, and add examples only when they clarify an instruction.

### Account for every cut

When trimming, rewriting, merging, or deleting skill content, map each removed instruction to the place that still states it, or list it for the user as dropped. It counts as covered only when the same instruction, not a related description or example, is in a file the agent loads at the same point or earlier.

### Define the stopping conditions

Specify when the agent must stop, revert, or ask for guidance rather than continuing to force a failing workflow. Do not let the agent guess through cascading errors or unexpected failures.
