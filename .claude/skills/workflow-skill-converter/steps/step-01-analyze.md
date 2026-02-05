---
name: step-01-analyze
description: Analyze skill structure, identify frontmatter, body, and agent references
prev_step: steps/step-00-init.md
next_step: steps/step-02-convert-agents.md
---

# Step 1: Analyze Skill

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER convert anything yet - analysis only
- 🛑 NEVER skip reading referenced files (steps/, agents)
- ✅ ALWAYS identify ALL agent references
- 📋 YOU ARE AN ANALYZER, not a converter
- 💬 FOCUS on understanding structure completely
- 🚫 FORBIDDEN to generate Kilocode output in this step

## EXECUTION PROTOCOLS:

- 🎯 Read SKILL.md completely
- 💾 Document all findings before proceeding
- 📖 Follow all file references to map dependencies
- 🚫 FORBIDDEN to load step-02 until analysis complete

## CONTEXT BOUNDARIES:

- Variables from step-00: `{skill_path}`, `{skill_name}`, `{auto_mode}`, `{save_mode}`, `{output_dir}`
- This step produces: `{skill_structure}`, `{referenced_agents}`
- Don't convert - just analyze

## YOUR TASK:

Read and analyze the Claude Code skill to understand its complete structure and dependencies.

---

<available_state>
From step-00-init:
- `{skill_path}` - Full path to skill directory
- `{skill_name}` - Skill name
- `{auto_mode}` - Skip confirmations
- `{save_mode}` - Save outputs
- `{output_dir}` - Output directory (if save_mode)
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Read SKILL.md

Read `{skill_path}/SKILL.md` completely.

**Parse YAML frontmatter:**
```yaml
name: {string}
description: {string}
argument-hint: {string|null}
disable-model-invocation: {boolean|null}
user-invocable: {boolean|null}
allowed-tools: {string|null}
model: {string|null}
context: {string|null}  # "fork" indicates subagent execution
agent: {string|null}    # Subagent type when context: fork
hooks: {object|null}
```

**Parse body content:**
- Main instructions
- `$ARGUMENTS` usage locations
- `!`command`` dynamic context injections
- References to other files (steps/, references/)

Store in `{skill_structure}`:
```yaml
frontmatter:
  name: ...
  description: ...
  # ... all fields
body:
  raw: "full body content"
  has_arguments: boolean
  argument_locations: ["line 5", "line 12"]
  has_dynamic_context: boolean
  dynamic_context_commands: ["git status", "git log"]
  file_references: ["steps/step-01.md", "references/api.md"]
```

### 2. Identify Agent References

Search for subagent patterns in the skill:

**Pattern 1: `context: fork` with `agent` field**
```yaml
context: fork
agent: Explore  # or custom agent name
```
→ Add agent to `{referenced_agents}`

**Pattern 2: Task tool invocations in body**
```markdown
Task(subagent_type=code-reviewer)
subagent_type: "snipper"
```
→ Add each unique subagent_type to `{referenced_agents}`

**Pattern 3: References to .claude/agents/**
```markdown
~/.claude/agents/my-agent.md
.claude/agents/custom-agent.md
```
→ Add each agent file to `{referenced_agents}`

**Pattern 4: Built-in agent references**
```
Explore, Plan, general-purpose, Bash, Snipper
```
→ Mark these as "built-in" (may need mode creation)

### 3. Read Referenced Files

For each file in `{skill_structure}.body.file_references`:
- Read the file
- Note its purpose (step file, reference doc, asset)
- Check for additional agent references

For workflow skills with `steps/` directory:
- Read ALL step files
- Aggregate agent references from all steps

### 4. Classify Agent References

For each agent in `{referenced_agents}`:

**Built-in agents** (Explore, Plan, general-purpose, Bash):
→ Mark as `needs_mode: true` with recommended Kilocode groups

**Custom agents** (files in .claude/agents/):
→ Read the agent file
→ Extract: name, description, tools, model, body
→ Mark as `needs_conversion: true`

**Unknown references:**
→ Mark as `needs_clarification: true`

Store in `{referenced_agents}`:
```yaml
- name: "code-reviewer"
  type: "custom"
  path: "~/.claude/agents/code-reviewer.md"
  needs_conversion: true
  structure:
    name: ...
    description: ...
    tools: [Read, Grep, Glob]
    body: "..."

- name: "Explore"
  type: "built-in"
  needs_mode: true
  recommended_groups: [read]
```

### 5. Compile Analysis Summary

Create comprehensive analysis:

```markdown
## Skill Analysis: {skill_name}

### Frontmatter Fields
| Field | Value | Kilocode Support |
|-------|-------|------------------|
| name | {value} | ✅ Supported |
| description | {value} | ✅ Supported |
| argument-hint | {value} | ⚠️ Document in description |
| allowed-tools | {value} | ❌ Not supported |
| model | {value} | ❌ Not supported |
| context | {value} | ⚠️ Convert to mode |
| hooks | {value} | ❌ Not supported |

### Body Content
- **Has $ARGUMENTS:** {yes/no} - Locations: {list}
- **Has dynamic context:** {yes/no} - Commands: {list}
- **File references:** {count} files

### Agent References
| Agent | Type | Needs Conversion |
|-------|------|------------------|
| {name} | {type} | {yes/no} |

### Conversion Complexity
- **Simple:** No agents, no dynamic context
- **Medium:** 1-2 agents, simple dynamic context
- **Complex:** Multiple agents, workflow steps, heavy dynamic context
```

**If `{save_mode}` = true:**
Append analysis to `{output_dir}/01-analyze.md`

### 6. Present Analysis

Display the analysis summary.

**If `{auto_mode}` = true:**
→ Proceed to step-02

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Analysis"
    question: "Skill analysis complete. How to proceed with {count} agent references?"
    options:
      - label: "Convert all agents (Recommended)"
        description: "Create Kilocode modes for all referenced agents"
      - label: "Skip agent conversion"
        description: "Only convert the skill, reference existing modes"
      - label: "Review analysis"
        description: "I want to review the details first"
    multiSelect: false
```

---

## SUCCESS METRICS:

✅ SKILL.md completely read and parsed
✅ All frontmatter fields identified
✅ All $ARGUMENTS locations found
✅ All dynamic context commands identified
✅ All referenced files read
✅ All agent references catalogued
✅ Conversion complexity assessed

## FAILURE MODES:

❌ Missing frontmatter parsing
❌ Not finding agent references in step files
❌ Skipping referenced file analysis
❌ Incomplete agent classification
❌ **CRITICAL**: Starting conversion before analysis complete

## ANALYZE PROTOCOLS:

- Read ALL files, not just SKILL.md
- Workflow skills have dependencies in steps/
- Built-in agents still need mode creation for Kilocode
- Track all dynamic context for documentation

---

## NEXT STEP:

After analysis confirmation, load `./step-02-convert-agents.md`

<critical>
Remember: Analysis is ONLY about understanding - don't convert anything yet!
Document everything thoroughly for reliable conversion.
</critical>
