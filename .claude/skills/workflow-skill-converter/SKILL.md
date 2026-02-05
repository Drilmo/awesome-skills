---
name: workflow-skill-converter
description: Convert Claude Code skills to Kilocode skills with full agent-to-mode conversion. Use when migrating skills from Claude Code to Kilocode, converting workflow skills, or transforming agents to modes. Supports -a flag for automatic conversion.
argument-hint: "[-a] [-s] <skill-path or skill-name>"
---

<objective>
Convert Claude Code skills (.claude/skills/) to Kilocode skills (.kilocode/skills/) with reliable, validated conversions. Automatically detects and converts referenced subagents to Kilocode modes.
</objective>

<quick_start>
**Basic usage:**
```bash
/workflow-skill-converter git-commit
```

**Automatic mode (no questions):**
```bash
/workflow-skill-converter -a workflow-apex
```

**With output saving:**
```bash
/workflow-skill-converter -a -s utils-refactor
```
</quick_start>

<parameters>
<flags>
| Short | Long | Description |
|-------|------|-------------|
| `-a` | `--auto` | Auto mode: skip confirmations, use defaults |
| `-s` | `--save` | Save mode: output conversion logs to `.claude/output/skill-converter/` |
</flags>

<input>
Path to skill or skill name from `~/.claude/skills/`
</input>

<examples>
```bash
# Convert by name (looks in ~/.claude/skills/)
/workflow-skill-converter git-commit

# Convert by path
/workflow-skill-converter /path/to/my-skill

# Auto mode - no questions
/workflow-skill-converter -a workflow-apex

# Save conversion logs
/workflow-skill-converter -a -s meta-hooks-creator
```
</examples>
</parameters>

<state_variables>
**Persist throughout all steps:**

| Variable | Type | Description |
|----------|------|-------------|
| `{skill_path}` | string | Full path to Claude Code skill |
| `{skill_name}` | string | Skill name (directory name) |
| `{auto_mode}` | boolean | Skip confirmations |
| `{save_mode}` | boolean | Save outputs to files |
| `{output_dir}` | string | Output directory (if save_mode) |
| `{skill_structure}` | object | Parsed skill frontmatter and body |
| `{referenced_agents}` | list | Subagents found in skill |
| `{converted_modes}` | list | Modes created from agents |
| `{kilocode_skill}` | object | Final converted skill |
| `{destination}` | string | Where to save Kilocode skill |
</state_variables>

<workflow>
**Conversion flow:**
```
step-00-init: Parse flags, locate skill
     ↓
step-01-analyze: Read skill, identify structure, find agent references
     ↓
step-02-convert-agents: Convert referenced agents to modes (if any)
     ↓
step-03-convert-skill: Convert skill frontmatter and body
     ↓
step-04-validate: Validate conversion completeness
     ↓
step-05-save: Save to Kilocode location
```
</workflow>

<entry_point>
**FIRST ACTION:** Load `steps/step-00-init.md`
</entry_point>

<step_files>
| Step | File | Purpose |
|------|------|---------|
| 00 | `steps/step-00-init.md` | Parse flags, locate skill, setup state |
| 01 | `steps/step-01-analyze.md` | Analyze skill structure, find agent refs |
| 02 | `steps/step-02-convert-agents.md` | Convert agents to Kilocode modes |
| 03 | `steps/step-03-convert-skill.md` | Convert skill to Kilocode format |
| 04 | `steps/step-04-validate.md` | Validate conversion completeness |
| 05 | `steps/step-05-save.md` | Save and report results |
</step_files>

<execution_rules>
- **Load one step at a time** - progressive loading
- **Persist state variables** across steps
- **Follow next_step directive** at end of each step
- **Save outputs** if `{save_mode}` = true
- **Skip AskUserQuestion** if `{auto_mode}` = true
</execution_rules>

<references>
- [references/frontmatter-mapping.md](references/frontmatter-mapping.md) - Field conversion reference
- [references/tool-to-group-mapping.md](references/tool-to-group-mapping.md) - Claude tools to Kilocode groups
- [references/conversion-patterns.md](references/conversion-patterns.md) - Common conversion patterns
</references>

<success_criteria>
- Skill frontmatter correctly converted
- Body content adapted for Kilocode
- All referenced agents converted to modes
- Dynamic context documented as limitations
- Converted skill validates against Kilocode schema
</success_criteria>
