---
name: step-02-convert-agents
description: Convert referenced Claude Code agents to Kilocode modes
prev_step: steps/step-01-analyze.md
next_step: steps/step-03-convert-skill.md
---

# Step 2: Convert Agents to Modes

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER skip an agent that needs conversion
- 🛑 NEVER create incomplete modes
- ✅ ALWAYS include roleDefinition from agent body
- ✅ ALWAYS map tools to groups correctly
- 📋 YOU ARE A CONVERTER, converting agents to modes
- 💬 FOCUS on agent-to-mode conversion only
- 🚫 FORBIDDEN to convert the main skill yet

## EXECUTION PROTOCOLS:

- 🎯 Convert each agent to a complete Kilocode mode
- 💾 Store converted modes in `{converted_modes}`
- 📖 Use tool-to-group mapping from references
- 🚫 FORBIDDEN to load step-03 until all agents converted

## CONTEXT BOUNDARIES:

- Variables from step-01: `{skill_structure}`, `{referenced_agents}`
- This step produces: `{converted_modes}` list
- Reference: `references/tool-to-group-mapping.md`

## YOUR TASK:

Convert all referenced Claude Code agents/subagents into Kilocode custom modes.

---

<available_state>
From previous steps:
- `{skill_path}` - Full path to skill
- `{skill_name}` - Skill name
- `{auto_mode}` - Skip confirmations
- `{save_mode}` - Save outputs
- `{output_dir}` - Output directory
- `{skill_structure}` - Parsed skill content
- `{referenced_agents}` - List of agents to convert
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Check if Conversion Needed

**If `{referenced_agents}` is empty:**
→ Skip to step 6 (no agents to convert)
→ Set `{converted_modes}` = []

**If agents exist:**
→ Continue to step 2

### 2. Load Tool-to-Group Mapping

Reference: See `references/tool-to-group-mapping.md`

**Core mapping:**
| Claude Code Tool | Kilocode Group |
|-----------------|----------------|
| Read, Glob, Grep | `read` |
| Edit, Write, MultiEdit, NotebookEdit | `edit` |
| Bash | `command` |
| WebFetch, WebSearch | `browser` |
| MCP, Task | `mcp` |

### 3. Convert Each Agent

For each agent in `{referenced_agents}`:

**3.1 Read Agent Source**

For custom agents:
→ Read `{agent.path}` (e.g., `~/.claude/agents/code-reviewer.md`)

For built-in agents:
→ Use predefined templates (see below)

**3.2 Parse Agent Structure**

```yaml
frontmatter:
  name: {string}
  description: {string}
  tools: {comma-separated list or null}
  model: {string or null}
body: {markdown content}
```

**3.3 Map Tools to Groups**

```
Input tools: "Read, Grep, Glob, Bash"
Output groups: ["read", "command"]
```

Apply mapping rules:
- Read/Glob/Grep → `read`
- Edit/Write → `edit`
- Bash → `command`
- WebFetch/WebSearch → `browser`
- MCP/Task → `mcp`
- No tools specified → inherit all groups

**3.4 Extract Role Definition**

From agent body, extract:
- Role statements ("You are a...")
- Expertise descriptions
- Core responsibilities

This becomes `roleDefinition`.

**3.5 Extract Custom Instructions**

From agent body, extract:
- Rules and constraints
- Process guidelines
- Output requirements

This becomes `customInstructions`.

**3.6 Generate Mode YAML**

```yaml
- slug: {agent-name-kebab-case}
  name: "{emoji} {Agent Name Title Case}"
  description: {from agent description}
  roleDefinition: |
    {extracted role definition}
  groups:
    - {group1}
    - {group2}
  customInstructions: |
    {extracted instructions}
```

**3.7 Infer Emoji**

Based on agent purpose:
| Purpose | Emoji |
|---------|-------|
| Code review | 🔎 |
| Testing | 🧪 |
| Documentation | 📝 |
| Security | 🔒 |
| Performance | ⚡ |
| Debugging | 🐛 |
| Research/Explore | 🔍 |
| Planning | 📋 |
| General coding | 💻 |

### 4. Handle Built-in Agents

For built-in Claude Code agents, use these templates:

**Explore → Codebase Explorer**
```yaml
- slug: explore-codebase
  name: "🔍 Codebase Explorer"
  description: Read-only exploration of codebase structure and patterns
  roleDefinition: |
    You are a codebase explorer. Find patterns, analyze structure,
    and discover relevant files. Do NOT modify any files.
  groups:
    - read
  whenToUse: When you need to understand existing code patterns
```

**Plan → Architect**
```yaml
- slug: architect
  name: "📋 Architect"
  description: Design and plan implementation approaches
  roleDefinition: |
    You are a software architect. Analyze requirements, design
    solutions, and create implementation plans. Focus on structure.
  groups:
    - read
  whenToUse: When planning implementations or designing architecture
```

**Snipper → Fast Coder**
```yaml
- slug: snipper
  name: "⚡ Snipper"
  description: Fast, focused code modifications
  roleDefinition: |
    You are a rapid code modification specialist. Make quick,
    focused changes. Follow existing patterns. Minimal output.
  groups:
    - read
    - edit
  customInstructions: |
    - Make minimal changes
    - Follow existing code style
    - No explanations unless asked
```

**general-purpose → Full Stack**
```yaml
- slug: full-stack
  name: "💻 Full Stack"
  description: General purpose coding with all capabilities
  roleDefinition: |
    You are a full-stack developer with complete access to
    read, edit, and execute commands.
  groups:
    - read
    - edit
    - command
```

### 5. Store Converted Modes

Add each converted mode to `{converted_modes}`:
```yaml
- slug: code-reviewer
  yaml: |
    {complete mode YAML}
  source_agent: "~/.claude/agents/code-reviewer.md"
```

**If `{save_mode}` = true:**
Append to `{output_dir}/02-convert-agents.md`:
```markdown
## Converted Modes

### {mode.slug}
Source: {mode.source_agent}

```yaml
{mode.yaml}
```
```

### 6. Present Conversion Results

```markdown
## Agent to Mode Conversion Complete

**Modes Created:** {count}

| Agent | Mode Slug | Groups |
|-------|-----------|--------|
| {agent} | {slug} | {groups} |

{if any agents skipped}
**Skipped:** {list with reasons}
```

**If `{auto_mode}` = true:**
→ Proceed to step-03

**If `{auto_mode}` = false:**
Use AskUserQuestion:
```yaml
questions:
  - header: "Modes"
    question: "Created {count} modes from agents. Continue with skill conversion?"
    options:
      - label: "Continue (Recommended)"
        description: "Proceed to convert the main skill"
      - label: "Review modes"
        description: "I want to review the generated modes first"
      - label: "Modify modes"
        description: "I want to change some mode settings"
    multiSelect: false
```

---

## SUCCESS METRICS:

✅ All agents in `{referenced_agents}` processed
✅ Tools correctly mapped to groups
✅ roleDefinition extracted from agent body
✅ Appropriate emoji assigned
✅ Valid Kilocode mode YAML generated
✅ `{converted_modes}` populated

## FAILURE MODES:

❌ Missing agent file (file not found)
❌ Incorrect tool-to-group mapping
❌ Empty roleDefinition
❌ Invalid YAML syntax
❌ Missing required fields (slug, name, groups)
❌ **CRITICAL**: Skipping agents without user confirmation

## AGENT CONVERSION PROTOCOLS:

- Always read the full agent file
- Extract meaningful content for roleDefinition
- Don't just copy - adapt for Kilocode context
- Built-in agents need templates, not conversion

---

## NEXT STEP:

After mode conversion confirmation, load `./step-03-convert-skill.md`

<critical>
Remember: Every referenced agent MUST become a Kilocode mode.
The skill will reference these modes - incomplete conversion breaks the skill!
</critical>
