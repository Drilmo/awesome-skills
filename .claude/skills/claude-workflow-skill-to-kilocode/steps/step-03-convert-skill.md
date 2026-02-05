---
name: step-03-convert-skill
description: Convert Claude Code skill to Kilocode skill format
prev_step: steps/step-02-convert-agents.md
next_step: steps/step-04-validate.md
---

# Step 3: Convert Skill

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER lose content from the original skill
- 🛑 NEVER leave $ARGUMENTS or !`command` unconverted
- ✅ ALWAYS document unsupported features as notes
- ✅ ALWAYS update agent references to mode references
- 📋 YOU ARE A SKILL CONVERTER, transforming format
- 💬 FOCUS on accurate conversion with documentation
- 🚫 FORBIDDEN to validate yet - just convert

## EXECUTION PROTOCOLS:

- 🎯 Convert frontmatter field by field
- 💾 Transform body content for Kilocode
- 📖 Reference mapping from `references/frontmatter-mapping.md`
- 🚫 FORBIDDEN to load step-04 until conversion complete

## CONTEXT BOUNDARIES:

- Variables from step-02: `{converted_modes}`
- This step produces: `{kilocode_skill}` complete conversion
- Use mappings from references/

## YOUR TASK:

Convert the Claude Code skill structure to Kilocode skill format.

---

<available_state>
From previous steps:
- `{skill_path}`, `{skill_name}`
- `{auto_mode}`, `{save_mode}`, `{output_dir}`
- `{skill_structure}` - Parsed Claude Code skill
- `{referenced_agents}` - Original agent refs
- `{converted_modes}` - Converted Kilocode modes
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Convert Frontmatter

**1.1 Direct Mappings**

| Claude Code | Kilocode | Action |
|-------------|----------|--------|
| `name` | `name` | Copy directly (must match directory) |
| `description` | `description` | Copy, enhance with trigger phrases |

**1.2 Unsupported Fields (Document)**

| Field | Kilocode Support | Conversion Action |
|-------|------------------|-------------------|
| `argument-hint` | ❌ Not supported | Add to description: "Usage: ..." |
| `disable-model-invocation` | ❌ Not supported | Note in description if manual-only |
| `user-invocable` | ❌ Not supported | Remove |
| `allowed-tools` | ❌ Not supported | Document in body as requirements |
| `model` | ❌ Not supported | Note in comments |
| `context: fork` | ❌ Not supported | Convert to mode reference |
| `agent` | ❌ Not supported | Reference converted mode |
| `hooks` | ❌ Not supported | Document as manual steps |

**1.3 Generate Kilocode Frontmatter**

```yaml
---
name: {skill_name}
description: {enhanced_description}
---
```

Enhancement patterns:
- Add "Use when..." trigger phrases
- Include usage pattern if had argument-hint
- Mention manual invocation: "Tell AI: use the {name} skill to..."

### 2. Convert Body Content

**2.1 Handle $ARGUMENTS**

Find all `$ARGUMENTS`, `$ARGUMENTS[N]`, `$N` patterns.

Convert to instructions:
```markdown
# Before (Claude Code)
Process file $ARGUMENTS[0] with format $ARGUMENTS[1]

# After (Kilocode)
Process the specified file with the specified format.

## Usage
Tell the AI: "use this skill on [filename] with [format]"
```

**2.2 Handle Dynamic Context**

Find all `!`command`` patterns.

Convert to explicit steps:
```markdown
# Before (Claude Code)
## Context
- Git status: !`git status`
- Branch: !`git branch --show-current`

# After (Kilocode)
## Context Gathering
First, gather context by running:
1. `git status` - to see current changes
2. `git branch --show-current` - to identify current branch
```

**2.3 Update Agent References**

Replace Task tool references with mode references:
```markdown
# Before (Claude Code)
Launch Task(subagent_type=code-reviewer) to analyze code

# After (Kilocode)
Switch to **Code Reviewer** mode to analyze code

Note: Requires code-reviewer mode configured in .kilocodemodes
```

Replace `context: fork` + `agent` references:
```markdown
# Before (skill with context: fork, agent: Explore)
This skill runs in isolated Explore context

# After
This skill should be run in **Codebase Explorer** mode.
Ensure the explore-codebase mode is configured.
```

**2.4 Convert Workflow Steps**

For skills with `steps/` directory:

Option A: Inline steps
→ Combine all step content into single SKILL.md
→ Convert step references to numbered sections

Option B: Preserve structure
→ Copy steps/ to Kilocode skill directory
→ Update any agent references in each step

**2.5 Handle Hooks**

Convert hooks to documented manual steps:
```markdown
# Before (Claude Code)
hooks:
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "npm run lint"

# After (Kilocode)
## Post-Edit Steps
After editing files, always run:
- `npm run lint` to check for issues
```

### 3. Assemble Kilocode Skill

Combine all converted elements:

```yaml
---
name: {skill_name}
description: {enhanced_description}
---

# {Skill Title}

{Converted body content}

## Usage
{How to invoke this skill}

## Requirements
{Tool requirements, mode dependencies}

## Notes
{Conversion notes, limitations}
- Converted from Claude Code skill
- Original features not supported:
  - {list unsupported features}
```

Store in `{kilocode_skill}`:
```yaml
frontmatter:
  name: ...
  description: ...
body: |
  {complete converted body}
notes:
  - "Original had $ARGUMENTS for X"
  - "Dynamic context converted to steps"
mode_dependencies:
  - slug: code-reviewer
  - slug: explore-codebase
```

### 4. Handle Supporting Files

**4.1 References directory**
- Copy `references/` to Kilocode skill
- Update any internal links

**4.2 Scripts directory**
- Copy `scripts/` if present
- Note: Kilocode may not have same script execution context

**4.3 Assets directory**
- Copy `assets/` directly
- No conversion needed

### 5. Document Limitations

Create limitations section:
```markdown
## Conversion Notes

This skill was converted from Claude Code. The following features
are not supported in Kilocode:

1. **Dynamic context injection** (`!`command``)
   - Original commands: {list}
   - Converted to: explicit workflow steps

2. **Argument passing** (`$ARGUMENTS`)
   - Original usage: {description}
   - Converted to: manual invocation guide

3. **Tool restrictions** (`allowed-tools`)
   - Original: {tool list}
   - Note: Kilocode doesn't restrict tools per skill

4. **Model selection** (`model`)
   - Original: {model}
   - Note: Kilocode uses global model setting

{if had hooks}
5. **Hooks**
   - Original hooks converted to manual steps
```

**If `{save_mode}` = true:**
Append to `{output_dir}/03-convert-skill.md`

### 6. Present Conversion

Display converted skill preview.

**If `{auto_mode}` = true:**
→ Proceed to step-04

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Conversion"
    question: "Skill converted. Review and continue to validation?"
    options:
      - label: "Validate conversion (Recommended)"
        description: "Check the conversion is complete and correct"
      - label: "Review conversion"
        description: "I want to see the full converted skill first"
      - label: "Modify conversion"
        description: "I want to change something"
    multiSelect: false
```

---

## SUCCESS METRICS:

✅ All frontmatter fields processed
✅ All $ARGUMENTS converted to instructions
✅ All dynamic context converted to steps
✅ All agent references updated to modes
✅ Supporting files handled
✅ Limitations documented
✅ Valid Kilocode skill structure

## FAILURE MODES:

❌ Lost content from original skill
❌ Unconverted $ARGUMENTS patterns
❌ Unconverted !`command` patterns
❌ Missing mode dependencies in notes
❌ Invalid YAML frontmatter
❌ **CRITICAL**: Agent references not updated to modes

## CONVERSION PROTOCOLS:

- Preserve ALL original functionality intent
- Document what couldn't be converted
- Make converted skill self-documenting
- Include mode dependencies clearly

---

## NEXT STEP:

After conversion confirmation, load `./step-04-validate.md`

<critical>
Remember: The goal is a WORKING Kilocode skill that preserves the
original skill's intent. Document all limitations clearly!
</critical>
