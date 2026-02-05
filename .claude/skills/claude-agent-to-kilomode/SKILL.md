---
name: claude-agent-to-kilomode
description: Convert Claude Code agents to Kilocode modes. Use when user wants to "convert agent", "agent to mode", "migrate agent to kilocode", or transform Claude Code agents into Kilocode custom modes. Supports -a flag for automatic conversion.
argument-hint: "[-a] <agent-path or agent-name>"
---

# Claude Agent to Kilocode Mode Converter

Convert Claude Code agents (.md files) to Kilocode custom modes (.kilocodemodes YAML).

## Argument Parsing

Parse arguments to detect:
- `-a` flag: Auto mode (no questions, convert directly)
- Path or name: Agent file path or agent name from `~/.claude/agents/`

## Workflow

### Step 1: Locate and Read Agent

If argument is a path, read directly. Otherwise, look in `~/.claude/agents/{name}.md`.

### Step 2: Parse Agent Structure

Extract from frontmatter:
- `name` → `slug` (kebab-case)
- `description` → `description` and `whenToUse`
- `tools` → `groups` (see mapping below)
- `model` → note for user (not directly mapped)

Extract from body:
- Role/personality content → `roleDefinition`
- Instructions/constraints → `customInstructions`

### Step 3: Map Tools to Groups

| Claude Code Tool | Kilocode Group |
|-----------------|----------------|
| Read, Glob, Grep | `read` |
| Edit, Write, MultiEdit, NotebookEdit | `edit` |
| Bash | `command` |
| WebFetch, WebSearch | `browser` |
| MCP, Task | `mcp` |

### Step 4: Generate Mode (depends on `-a` flag)

**If `-a` flag (Auto Mode):**
- Convert immediately using extracted data
- Infer emoji from agent purpose
- Output complete YAML

**If NO `-a` flag (Interactive Mode):**
Ask about unclear aspects:
1. Emoji preference for mode name?
2. File restrictions needed for edit group?
3. Any additional custom instructions?
4. Where to save? (project `.kilocodemodes` or global `custom_modes.yaml`)

## Output Format

```yaml
customModes:
  - slug: agent-slug
    name: "🎯 Agent Name"
    description: Short description from agent
    roleDefinition: |
      Converted from agent body content.
      Includes role and expertise description.
    groups:
      - read
      - edit
      - command
    whenToUse: From agent description or capabilities
    customInstructions: |
      Converted constraints and instructions from agent.
```

## Conversion Examples

### Input (Claude Code Agent)
```markdown
---
name: snipper
description: Fast code modification agent
tools: Read, Edit
model: haiku
---
You are a rapid code modification specialist.
## Rules
- Follow existing code style
- Make minimal changes
```

### Output (Kilocode Mode)
```yaml
customModes:
  - slug: snipper
    name: "⚡ Snipper"
    description: Fast code modification agent
    roleDefinition: |
      You are a rapid code modification specialist.
    groups:
      - read
      - edit
    customInstructions: |
      - Follow existing code style
      - Make minimal changes
```

## Reference

See [references/tool-mapping.md](references/tool-mapping.md) for complete tool-to-group mapping and edge cases.
