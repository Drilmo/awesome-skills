---
name: step-05-save
description: Save converted skill and modes to output directory
prev_step: steps/step-04-validate.md
---

# Step 5: Save Conversion

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER overwrite existing output directory without confirmation (unless auto_mode)
- 🛑 NEVER save invalid content
- ✅ ALWAYS create the complete directory structure
- ✅ ALWAYS save modes as individual files
- 📋 YOU ARE A SAVER, creating the deliverable package
- 💬 FOCUS on correct file structure and placement
- 🚫 FORBIDDEN to modify content - only save

## EXECUTION PROTOCOLS:

- 🎯 Create output directory in current working directory
- 💾 Create `skills/` and `modes/` subdirectories
- 📖 Save each mode as a separate YAML file
- 🚫 FORBIDDEN to skip any converted content

## CONTEXT BOUNDARIES:

- Variables from step-04: `{validation_result}` must be PASSED
- This step produces: self-contained output directory
- No more modifications to content

## YOUR TASK:

Create a self-contained output directory with the converted skill and modes.

---

<available_state>
From previous steps:
- `{skill_name}` - Skill being converted
- `{skill_path}` - Original skill location
- `{auto_mode}`, `{save_mode}`, `{output_dir}`
- `{kilocode_skill}` - Converted skill content
- `{converted_modes}` - Converted modes list
- `{validation_result}` - Should be PASSED
</available_state>

---

## OUTPUT STRUCTURE

The deliverable is a directory in the current working directory:

```
{skill_name}/
├── skills/
│   └── {skill_name}/
│       ├── SKILL.md
│       ├── references/     (if original had references)
│       ├── scripts/        (if original had scripts)
│       └── assets/         (if original had assets)
└── modes/
    ├── {mode-slug-1}.yaml
    ├── {mode-slug-2}.yaml
    └── ...
```

---

## EXECUTION SEQUENCE:

### 1. Determine Output Location

Set `{output_root}` = `./{skill_name}/`

**Check if directory already exists:**

**If `{auto_mode}` = true AND directory exists:**
→ Create backup: `{skill_name}.backup-{timestamp}/`
→ Proceed with creation

**If `{auto_mode}` = false AND directory exists:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Overwrite"
    question: "Directory ./{skill_name}/ already exists. Overwrite?"
    options:
      - label: "Overwrite (backup created)"
        description: "Replace existing directory, keep backup"
      - label: "Choose different name"
        description: "Save with a different directory name"
      - label: "Cancel"
        description: "Don't save"
    multiSelect: false
```

### 2. Create Directory Structure

```bash
mkdir -p {output_root}/skills/{skill_name}
mkdir -p {output_root}/modes
```

### 3. Save Skill Files

**3.1 Write SKILL.md:**

```bash
write {output_root}/skills/{skill_name}/SKILL.md
```

Content: `{kilocode_skill.frontmatter}` + `{kilocode_skill.body}`

**3.2 Copy supporting directories:**

If original skill had `references/`:
```bash
cp -r {skill_path}/references/ {output_root}/skills/{skill_name}/references/
```
(Convert any references if they contain Claude Code specifics)

If original skill had `scripts/`:
```bash
cp -r {skill_path}/scripts/ {output_root}/skills/{skill_name}/scripts/
```

If original skill had `assets/`:
```bash
cp -r {skill_path}/assets/ {output_root}/skills/{skill_name}/assets/
```

If original skill had `steps/` AND steps were NOT inlined:
```bash
cp -r {skill_path}/steps/ {output_root}/skills/{skill_name}/steps/
```
(Convert any step files if they contain Claude Code specifics)

### 4. Save Mode Files

For each mode in `{converted_modes}`:

**4.1 Generate mode filename:**
`{mode.slug}.yaml`

**4.2 Write mode file:**

```bash
write {output_root}/modes/{mode.slug}.yaml
```

Content for each mode file:
```yaml
# Kilocode Mode: {mode.name}
# Converted from Claude Code agent: {mode.source_agent}
#
# To use: Add this content to .kilocodemodes or ~/.kilocode/custom_modes.yaml
# under the customModes: array

- slug: {mode.slug}
  name: "{mode.name}"
  description: {mode.description}
  roleDefinition: |
    {mode.roleDefinition}
  groups:
    - {group1}
    - {group2}
  whenToUse: {mode.whenToUse}
  customInstructions: |
    {mode.customInstructions}
```

**4.3 Create combined modes file:**

Also create `{output_root}/modes/_all-modes.yaml` with all modes combined:
```yaml
# Combined Kilocode Modes
# Converted from Claude Code skill: {skill_name}
#
# Copy this content to .kilocodemodes or ~/.kilocode/custom_modes.yaml

customModes:
  {all converted_modes as YAML array}
```

### 5. Create README

Create `{output_root}/README.md`:

```markdown
# Converted Skill: {skill_name}

This directory contains a Claude Code skill converted to Kilocode format.

## Installation

### 1. Install the Skill

Copy the skill directory to your Kilocode skills location:

**Global (all projects):**
```bash
cp -r skills/{skill_name} ~/.kilocode/skills/
```

**Project-specific:**
```bash
cp -r skills/{skill_name} .kilocode/skills/
```

### 2. Install the Modes

{if converted_modes is not empty}
This skill requires the following Kilocode modes:

| Mode | Slug | File |
|------|------|------|
{for each mode}
| {mode.name} | `{mode.slug}` | `modes/{mode.slug}.yaml` |

**Option A: Copy all modes at once**
Merge `modes/_all-modes.yaml` content into your `.kilocodemodes` file.

**Option B: Copy individual modes**
Copy specific mode files from `modes/` directory.
{endif}

{if converted_modes is empty}
This skill has no mode dependencies.
{endif}

## Conversion Notes

**Original skill:** `{skill_path}`
**Converted:** {timestamp}

### Limitations

{list of documented limitations from kilocode_skill.notes}

## Testing

After installation, test the skill in Kilocode:
```
use the {skill_name} skill to [action]
```
```

### 6. Verify Output

**6.1 Check files exist:**

| Path | Exists |
|------|--------|
| `{output_root}/skills/{skill_name}/SKILL.md` | ✓/✗ |
| `{output_root}/modes/_all-modes.yaml` | ✓/✗ |
| `{output_root}/README.md` | ✓/✗ |

For each mode in `{converted_modes}`:
| `{output_root}/modes/{mode.slug}.yaml` | ✓/✗ |

**6.2 Validate YAML syntax:**

Read back saved files and verify:
- YAML parses correctly
- Required fields present
- No corruption during write

### 7. Generate Final Report

```markdown
# Conversion Complete: {skill_name}

## Output Location

📁 `./{skill_name}/`

## Files Created

### Skill
```
{skill_name}/skills/{skill_name}/
├── SKILL.md
{if references}├── references/ ({count} files){endif}
{if scripts}├── scripts/ ({count} files){endif}
{if assets}├── assets/ ({count} files){endif}
```

### Modes ({count})
```
{skill_name}/modes/
├── _all-modes.yaml (combined)
{for each mode}├── {mode.slug}.yaml{endfor}
```

| Slug | Name | Groups |
|------|------|--------|
{for each mode}| {slug} | {name} | {groups} |{endfor}

## Next Steps

1. **Review the output:**
   ```bash
   ls -la ./{skill_name}/
   cat ./{skill_name}/README.md
   ```

2. **Install to Kilocode:**
   ```bash
   cp -r ./{skill_name}/skills/{skill_name} ~/.kilocode/skills/
   ```

3. **Install modes (if any):**
   Merge `./{skill_name}/modes/_all-modes.yaml` into `.kilocodemodes`

4. **Test in Kilocode:**
   "use the {skill_name} skill"

## Conversion Notes

{list of documented limitations}

---
✅ Conversion completed at {timestamp}
```

**If `{save_mode}` = true:**
- Append to `.claude/output/skill-converter/{skill_name}/05-save.md`
- Update progress in `.claude/output/skill-converter/{skill_name}/00-context.md`

### 8. Display Completion

Present the final report to user.

**Success message:**
```
✅ Conversion complete!

Output directory: ./{skill_name}/

Contents:
- skills/{skill_name}/SKILL.md
{if modes}- modes/ ({count} mode files){endif}
- README.md (installation instructions)

Next: Review README.md for installation steps
```

---

## SUCCESS METRICS:

✅ Output directory created at `./{skill_name}/`
✅ Skill saved to `skills/{skill_name}/SKILL.md`
✅ All modes saved as individual YAML files
✅ Combined modes file created
✅ README with installation instructions created
✅ Supporting files copied
✅ Files verified readable
✅ Final report generated

## FAILURE MODES:

❌ Permission denied writing files
❌ Directory creation failed
❌ YAML syntax errors in output
❌ Missing supporting files
❌ Incomplete mode files
❌ **CRITICAL**: Saving without validation pass

## SAVE PROTOCOLS:

- Create complete directory structure first
- Save each mode as individual file for flexibility
- Also create combined file for convenience
- Include clear installation README
- Verify all writes by reading back

---

## WORKFLOW COMPLETE

The conversion workflow is now finished.

<critical>
Remember: A successful conversion produces a self-contained directory with:
1. `skills/{skill_name}/` - The converted Kilocode skill
2. `modes/` - Individual mode files + combined file
3. `README.md` - Clear installation instructions

User can review, modify, then copy to Kilocode locations!
</critical>
