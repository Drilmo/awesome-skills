---
name: step-00-init
description: Initialize workflow - parse flags, locate skill, setup state
next_step: steps/step-01-analyze.md
---

# Step 0: Initialization

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER skip flag parsing
- 🛑 NEVER proceed if skill not found
- ✅ ALWAYS validate skill path exists before continuing
- 📋 Parse ALL flags before locating skill
- 💬 FOCUS on initialization only - don't analyze skill yet
- 🚫 FORBIDDEN to read skill content in this step

## EXECUTION PROTOCOLS:

- 🎯 Parse flags first, then locate skill
- 💾 Create output structure if save_mode
- 📖 Initialize all state variables
- 🚫 FORBIDDEN to load step-01 until skill is located

## CONTEXT BOUNDARIES:

- Input: User arguments from $ARGUMENTS
- Output: State variables for subsequent steps
- No skill content loaded yet

## YOUR TASK:

Parse flags and arguments to locate the Claude Code skill to convert.

---

## DEFAULTS CONFIGURATION:

```yaml
auto_mode: false    # -a: Skip confirmations, use defaults
save_mode: false    # -s: Save outputs to .claude/output/skill-converter/
```

---

## INITIALIZATION SEQUENCE:

### 1. Parse Flags and Input

**Step 1: Load defaults from config above**

**Step 2: Parse user input:**
```
Enable flags (lowercase):
  -a → {auto_mode} = true
  -s → {save_mode} = true

Remainder (after removing flags) → {skill_input}
```

**Step 3: Determine skill location:**

If `{skill_input}` is an absolute path:
→ `{skill_path}` = `{skill_input}`

If `{skill_input}` is a relative path or name:
→ Look in `~/.claude/skills/{skill_input}/SKILL.md`
→ Then look in `.claude/skills/{skill_input}/SKILL.md`

**Step 4: Extract skill name:**
→ `{skill_name}` = directory name containing SKILL.md

### 2. Validate Skill Exists

**Check that `{skill_path}/SKILL.md` exists:**

**If exists:**
→ Continue to step 3

**If NOT exists:**
→ Report error: "Skill not found: {skill_input}"
→ List available skills in ~/.claude/skills/
→ **STOP** workflow

### 3. Initialize Output Paths

**Always set output root:**
Set `{output_root}` = `./{skill_name}/`

This is where the converted skill and modes will be saved (in step-05).

**If `{save_mode}` = true (conversion logs):**

```bash
mkdir -p .claude/output/skill-converter/{skill_name}/
```

Set `{logs_dir}` = `.claude/output/skill-converter/{skill_name}/`

Create `{logs_dir}/00-context.md`:
```markdown
# Skill Conversion: {skill_name}

**Created:** {timestamp}
**Source:** {skill_path}
**Output:** {output_root}
**Flags:** auto={auto_mode}, save={save_mode}

## Progress
| Step | Status | Timestamp |
|------|--------|-----------|
| 00-init | ✓ | {timestamp} |
| 01-analyze | ⏸ Pending | |
| 02-convert-agents | ⏸ Pending | |
| 03-convert-skill | ⏸ Pending | |
| 04-validate | ⏸ Pending | |
| 05-save | ⏸ Pending | |
```

### 4. Display Initialization Summary

```markdown
## Initialization Complete

**Skill:** {skill_name}
**Source:** {skill_path}
**Output:** {output_root} (will create: skills/ + modes/)
**Mode:** {auto_mode ? "Automatic" : "Interactive"}
**Logs:** {save_mode ? logs_dir : "None"}

Ready to analyze skill structure.
```

### 5. Confirm Start

**If `{auto_mode}` = true:**
→ Proceed directly to step-01

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Start"
    question: "Skill located. Ready to begin conversion?"
    options:
      - label: "Begin Conversion (Recommended)"
        description: "Start analyzing the skill"
      - label: "Cancel"
        description: "Don't convert this skill"
    multiSelect: false
```

---

## SUCCESS METRICS:

✅ Flags correctly parsed (-a, -s)
✅ Skill path validated and exists
✅ Skill name extracted
✅ Output folder created (if save_mode)
✅ State variables initialized

## FAILURE MODES:

❌ Skill not found at specified path
❌ No SKILL.md in skill directory
❌ Invalid flag combination
❌ Missing required skill input

## INIT PROTOCOLS:

- Always check both user (~/) and project (./) skill locations
- Report clear error if skill not found
- List available skills to help user

---

## NEXT STEP:

After confirmation, load `./step-01-analyze.md`

<critical>
Remember: Init is ONLY about locating the skill - don't read or analyze content yet!
</critical>
