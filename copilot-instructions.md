# Workspace Instructions

## Default Output Location for Skills, Prompts, and Agents

When a skill, prompt, or agent generates human-readable content (e.g., summaries, reports, docs, digests, analyses) and the user has **not** explicitly specified an output path, write the content to:

```
tmp/memory/<name>/
```

Examples:

- A `digest` skill → `tmp/memory/digest/`
- A `commit-helper` skill → `tmp/memory/commit-helper/`
- A `tdd-workflow` skill → `tmp/memory/tdd-workflow/`
- A `reviewer` agent → `tmp/memory/reviewer/`
- A `standup` prompt → `tmp/memory/standup/`

Rules:

- Use the primitive's own name as the folder name: the `SKILL.md` `name` field, the `.prompt.md` filename without extension, or the `.agent.md` filename without extension.
- Create the directory if it does not exist.
- This rule applies only to **outward-facing, human-readable** artifacts (markdown files, summaries, reports). It does NOT apply to source code edits or in-place file modifications.
- If the user specifies an explicit path, always honor it over this default.
