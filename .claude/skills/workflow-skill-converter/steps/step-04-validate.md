---
name: step-04-validate
description: Validate conversion completeness and correctness
prev_step: steps/step-03-convert-skill.md
next_step: steps/step-05-save.md
---

# Step 4: Validate Conversion

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER skip validation checks
- 🛑 NEVER approve invalid conversions
- ✅ ALWAYS check ALL validation criteria
- ✅ ALWAYS report issues clearly
- 📋 YOU ARE A VALIDATOR, ensuring quality
- 💬 FOCUS on finding problems before saving
- 🚫 FORBIDDEN to save if validation fails

## EXECUTION PROTOCOLS:

- 🎯 Run all validation checks systematically
- 💾 Document any issues found
- 📖 Verify against Kilocode requirements
- 🚫 FORBIDDEN to load step-05 if critical issues exist

## CONTEXT BOUNDARIES:

- Variables from step-03: `{kilocode_skill}`, `{converted_modes}`
- This step produces: `{validation_result}` with pass/fail
- No modifications - only validation

## YOUR TASK:

Validate that the converted skill and modes meet Kilocode requirements.

---

<available_state>
From previous steps:
- `{skill_name}` - Skill being converted
- `{auto_mode}`, `{save_mode}`, `{output_dir}`
- `{skill_structure}` - Original skill structure
- `{kilocode_skill}` - Converted skill
- `{converted_modes}` - Converted modes
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Validate Skill Frontmatter

**Check required fields:**

| Field | Requirement | Check |
|-------|-------------|-------|
| `name` | Present, matches directory name | ✓/✗ |
| `name` | Max 64 chars, lowercase, hyphens only | ✓/✗ |
| `description` | Present | ✓/✗ |
| `description` | Max 1024 chars | ✓/✗ |

**Record issues:**
```yaml
frontmatter_issues:
  - field: name
    issue: "Contains uppercase letters"
    severity: critical
```

### 2. Validate Skill Body

**2.1 Check for unconverted patterns:**

| Pattern | Should NOT exist | Found |
|---------|-----------------|-------|
| `$ARGUMENTS` | ❌ | {yes/no} |
| `$ARGUMENTS[N]` | ❌ | {yes/no} |
| `$0`, `$1`, etc. | ❌ | {yes/no} |
| `!`command`` | ❌ | {yes/no} |
| `${CLAUDE_SESSION_ID}` | ❌ | {yes/no} |

**2.2 Check agent references updated:**

For each agent in original `{referenced_agents}`:
→ Verify old reference replaced with mode reference
→ Check mode name matches `{converted_modes}`

**2.3 Check content completeness:**

| Check | Result |
|-------|--------|
| Original workflow preserved | ✓/✗ |
| Instructions are clear | ✓/✗ |
| Usage section present | ✓/✗ |
| Limitations documented | ✓/✗ |

### 3. Validate Converted Modes

For each mode in `{converted_modes}`:

**3.1 Required fields:**
| Field | Requirement | Check |
|-------|-------------|-------|
| `slug` | Present, kebab-case | ✓/✗ |
| `name` | Present, includes emoji | ✓/✗ |
| `description` | Present | ✓/✗ |
| `roleDefinition` | Present, not empty | ✓/✗ |
| `groups` | Present, valid values | ✓/✗ |

**3.2 Valid group values:**
- `read` ✓
- `edit` ✓
- `browser` ✓
- `command` ✓
- `mcp` ✓

**3.3 Mode YAML syntax:**
- Valid YAML structure
- Proper indentation
- No syntax errors

### 4. Cross-Reference Validation

**4.1 Mode dependencies satisfied:**

For each mode referenced in skill:
→ Check mode exists in `{converted_modes}`
→ Verify slug matches exactly

**4.2 No orphan modes:**

For each mode in `{converted_modes}`:
→ Verify it's referenced somewhere (skill or other modes)

**4.3 No circular dependencies:**

Check mode references don't create loops.

### 5. Compile Validation Report

```markdown
## Validation Report: {skill_name}

### Summary
| Category | Status | Issues |
|----------|--------|--------|
| Frontmatter | ✅/❌ | {count} |
| Body Content | ✅/❌ | {count} |
| Mode Validity | ✅/❌ | {count} |
| Cross-References | ✅/❌ | {count} |

### Overall: {PASS/FAIL}

### Issues Found

{if critical issues}
#### Critical (Must Fix)
1. {issue description}
   - Location: {where}
   - Fix: {how to fix}

{if warnings}
#### Warnings (Should Fix)
1. {warning description}

{if suggestions}
#### Suggestions (Optional)
1. {suggestion}
```

Store in `{validation_result}`:
```yaml
passed: boolean
critical_issues: []
warnings: []
suggestions: []
```

**If `{save_mode}` = true:**
Append to `{output_dir}/04-validate.md`

### 6. Handle Validation Result

**If validation PASSED:**

Display success summary.

**If `{auto_mode}` = true:**
→ Proceed to step-05

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Validation"
    question: "Validation passed! Ready to save the converted skill?"
    options:
      - label: "Save conversion (Recommended)"
        description: "Save skill and modes to Kilocode locations"
      - label: "Review first"
        description: "I want to review the full conversion"
    multiSelect: false
```

**If validation FAILED (critical issues):**

Display issues clearly.

**If `{auto_mode}` = true:**
→ Attempt auto-fix for common issues
→ Re-validate
→ If still fails: STOP and report

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Issues"
    question: "Validation found {count} critical issues. How to proceed?"
    options:
      - label: "Fix and retry"
        description: "Go back and fix the issues"
      - label: "Save anyway"
        description: "Save with known issues (not recommended)"
      - label: "Abort"
        description: "Cancel the conversion"
    multiSelect: false
```

---

## AUTO-FIX PATTERNS

Common issues that can be auto-fixed:

**1. Uppercase in name:**
→ Convert to lowercase

**2. Spaces in slug:**
→ Replace with hyphens

**3. Missing description:**
→ Generate from skill body

**4. Unconverted $ARGUMENTS:**
→ Replace with placeholder instructions

**5. Empty roleDefinition:**
→ Extract from mode description

---

## SUCCESS METRICS:

✅ All frontmatter fields validated
✅ All unconverted patterns detected
✅ All modes validated
✅ Cross-references verified
✅ Validation report generated
✅ Issues categorized by severity

## FAILURE MODES:

❌ Skipping validation checks
❌ Not detecting unconverted patterns
❌ Approving invalid mode YAML
❌ Missing cross-reference validation
❌ **CRITICAL**: Proceeding with critical issues

## VALIDATION PROTOCOLS:

- Be thorough - catch issues now, not after save
- Critical issues MUST be fixed
- Warnings should be reviewed
- Auto-fix only safe transformations

---

## NEXT STEP:

**If validation PASSED:**
Load `./step-05-save.md`

**If validation FAILED with critical issues:**
Return to step-03 to fix issues, or abort

<critical>
Remember: Validation protects against broken conversions.
NEVER skip validation - a broken skill wastes user time!
</critical>
