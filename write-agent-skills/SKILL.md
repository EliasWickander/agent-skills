---
name: write-agent-skills
description: "Creates new agent skills and improves existing skills. Use when users want to create a skill from scratch, edit a SKILL.md file, or improve an existing agent skill"
---

# Writing Skills

## Skill Structure

```
my-skill/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
```

## Name Requirements

### Format

- Max 64 characters; lowercase letters, numbers, and hyphens only
- No leading, trailing, or consecutive hyphens
- Match the parent directory name

### Content

- Use a specific name that clearly describes the task or capability (e.g. write-agent-skills)
- Avoid vague or overly generic names like helper, utils, or documents
- Use a consistent naming pattern within the skill collection

## Description Requirements

### Format

- Non-empty; max 1,024 characters
- No XML tags
- First sentence: What it does (in third person)
- Second sentence: Use when [specific triggers]

### Content

- Put the main use case first
- Name relevant tasks, tools, file types, or outputs
- Describe the user's intent, not the implementation
- Add ‘Do not use for…’ only when similar tasks could be confused

## Instruction Requirements

### Format

- Use short, direct instructions
- Use consistent terms
- Use numbered steps when order matters

### Content

- Focus on task-specific guidance; skip basic explanations
- State required inputs and expected outputs
- Make decision points and their conditions clear
- Give exact steps for fragile tasks; allow flexibility elsewhere
- Explain non-obvious rules briefly
- Add examples only when they clarify the instructions
- Add task-specific checks, safeguards, and error handling where needed
- Remove filler, duplication, and conflicting instructions

### Supporting Files

- Move content to references/ when SKILL.md nears 500 lines, or when only some tasks need it
- Keep essential instructions in SKILL.md; move lengthy reference material to references/ when useful
- Link to files in references/, scripts/, and assets/ and state when to read, run, or use them
- Link references directly from SKILL.md, not from other references
