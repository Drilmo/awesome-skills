---
name: step-05-save
description: Save converted skill and modes to Kilocode locations
prev_step: steps/step-04-validate.md
---

# Step 5: Save Conversion

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER overwrite without confirmation (unless auto_mode)
- 🛑 NEVER save invalid content
- ✅ ALWAYS create proper directory structure
- ✅ ALWAYS save modes before skill (dependency order)
- 📋 YOU ARE A SAVER, finalizing the conversion
- 💬 FOCUS on correct file placement
- 🚫 FORBIDDEN to modify content - only save

## EXECUTION PROTOCOLS:

- 🎯 Determine save locations based on user choice
- 💾 Create directories if needed
- 📖 Save in correct order: modes first, then skill
- 🚫 FORBIDDEN to skip backup of existing files

## CONTEXT BOUNDARIES:

- Variables from step-04: `{validation_result}` must be PASSED
- This step produces: saved files and final report
- No more modifications to content

## YOUR TASK:

Save the converted Kilocode skill and modes to their target locations.

---

<available_state>
From previous steps:
- `{skill_name}` - Skill being converted
- `{auto_mode}`, `{save_mode}`, `{output_dir}`
- `{kilocode_skill}` - Converted skill content
- `{converted_modes}` - Converted modes list
- `{validation_result}` - Should be PASSED
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Determine Save Locations

**1.1 Ask for destination (if not auto_mode):**

**If `{auto_mode}` = true:**
→ Use defaults:
  - Skill: `~/.kilocode/skills/{skill_name}/`
  - Modes: `~/.kilocode/custom_modes.yaml` (append)

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Skill Location"
    question: "Where should the converted skill be saved?"
    options:
      - label: "Global (~/.kilocode/skills/) (Recommended)"
        description: "Available in all projects"
      - label: "Project (.kilocode/skills/)"
        description: "Only in current project"
      - label: "Custom path"
        description: "I'll specify the path"
    multiSelect: false
  - header: "Mode Location"
    question: "Where should the converted modes be saved?"
    options:
      - label: "Global (~/.kilocode/custom_modes.yaml) (Recommended)"
        description: "Available in all projects"
      - label: "Project (.kilocodemodes)"
        description: "Only in current project"
    multiSelect: false
```

Set `{skill_destination}` and `{modes_destination}` based on response.

### 2. Check for Existing Files

**2.1 Check skill location:**

If `{skill_destination}/SKILL.md` exists:

**If `{auto_mode}` = true:**
→ Create backup: `SKILL.md.backup-{timestamp}`
→ Proceed with overwrite

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Overwrite"
    question: "Skill already exists at destination. Overwrite?"
    options:
      - label: "Overwrite (backup created)"
        description: "Replace existing skill, keep backup"
      - label: "Choose different name"
        description: "Save with a different skill name"
      - label: "Cancel"
        description: "Don't save"
    multiSelect: false
```

**2.2 Check modes file:**

If `{modes_destination}` exists:
→ Read existing modes
→ Check for slug conflicts
→ Plan merge strategy (append new modes)

### 3. Save Modes First

**3.1 Create/Update modes file:**

**For global location (`~/.kilocode/custom_modes.yaml`):**

If file exists:
```yaml
# Read existing
existing_modes = parse(file)

# Merge (new modes override conflicts)
for mode in {converted_modes}:
  existing_modes[mode.slug] = mode

# Write combined
write(file, existing_modes)
```

If file doesn't exist:
```yaml
# Create new
customModes:
  {converted_modes as YAML}
```

**For project location (`.kilocodemodes`):**

Same logic, different file.

**3.2 Report saved modes:**

```markdown
## Modes Saved

Location: {modes_destination}

| Mode Slug | Action |
|-----------|--------|
| {slug} | Created/Updated |
```

### 4. Save Skill

**4.1 Create skill directory:**

```bash
mkdir -p {skill_destination}
```

**4.2 Write SKILL.md:**

```bash
write {skill_destination}/SKILL.md
```

Content: `{kilocode_skill.frontmatter}` + `{kilocode_skill.body}`

**4.3 Copy supporting files:**

If original skill had `references/`:
→ Copy to `{skill_destination}/references/`

If original skill had `scripts/`:
→ Copy to `{skill_destination}/scripts/`

If original skill had `assets/`:
→ Copy to `{skill_destination}/assets/`

**4.4 Copy step files (for workflow skills):**

If original had `steps/` directory:
→ Option A: Already inlined in SKILL.md (skip)
→ Option B: Copy converted steps to `{skill_destination}/steps/`

### 5. Verify Save

**5.1 Check files exist:**

| File | Exists | Valid |
|------|--------|-------|
| `{skill_destination}/SKILL.md` | ✓/✗ | ✓/✗ |
| `{modes_destination}` | ✓/✗ | ✓/✗ |

**5.2 Validate YAML syntax:**

Read back saved files and verify:
- YAML parses correctly
- Required fields present
- No corruption during write

### 6. Generate Final Report

```markdown
# Conversion Complete: {skill_name}

## Summary

**Original:** {skill_path}
**Converted Skill:** {skill_destination}/SKILL.md
**Modes:** {modes_destination}

## Files Created

### Skill
- `{skill_destination}/SKILL.md`
{if references}
- `{skill_destination}/references/` ({count} files)
{if scripts}
- `{skill_destination}/scripts/` ({count} files)

### Modes ({count})
| Slug | Name | Groups |
|------|------|--------|
| {slug} | {name} | {groups} |

## Next Steps

1. **Test the skill:**
   Open Kilocode and try: "use the {skill_name} skill"

2. **Verify modes work:**
   Check modes are available in Kilocode mode selector

3. **Review limitations:**
   See conversion notes for features that couldn't be converted

## Conversion Notes

{list of documented limitations from kilocode_skill.notes}

---
Conversion completed at {timestamp}
```

**If `{save_mode}` = true:**
- Append to `{output_dir}/05-save.md`
- Update progress in `{output_dir}/00-context.md`

### 7. Display Completion

Present the final report to user.

**Success message:**
```
✅ Conversion complete!

Skill saved to: {skill_destination}
Modes saved to: {modes_destination}

Test it in Kilocode: "use the {skill_name} skill"
```

---

## SUCCESS METRICS:

✅ Skill saved to correct location
✅ All modes saved/merged correctly
✅ Supporting files copied
✅ Files verified readable
✅ Final report generated
✅ No file corruption

## FAILURE MODES:

❌ Permission denied writing files
❌ Directory creation failed
❌ Mode merge conflicts not handled
❌ File corruption during write
❌ Missing supporting files
❌ **CRITICAL**: Saving without validation pass

## SAVE PROTOCOLS:

- Always save modes before skill (dependency)
- Always create backups before overwrite
- Verify writes by reading back
- Report clear success/failure

---

## WORKFLOW COMPLETE

The conversion workflow is now finished.

<critical>
Remember: A successful conversion means:
1. Skill file exists and is valid YAML
2. All modes exist and are valid
3. User knows where files are
4. User knows how to test

If anything failed, provide clear error message and recovery steps!
</critical>
