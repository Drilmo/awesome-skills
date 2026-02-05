---
name: claude-skill-to-kilocode
description: Convert Claude Code skills to Kilocode skills. Use when user wants to "convert skill", "skill to kilocode", "migrate skill", or transform Claude Code skills into Kilocode skills format. Handles workflow skills with subagent conversion to modes. Supports -a flag for automatic conversion.
argument-hint: "[-a] <skill-path or skill-name>"
---

# Claude Code Skill to Kilocode Skill Converter

Convert Claude Code skills (.claude/skills/) to Kilocode skills (.kilocode/skills/).

## Argument Parsing

Parse arguments to detect:
- `-a` flag: Auto mode (no questions, convert directly)
- Path or name: Skill folder path or skill name from `~/.claude/skills/`

## Workflow

### Step 1: Locate and Read Skill

If argument is a path, read directly. Otherwise, look in `~/.claude/skills/{name}/SKILL.md`.

Read the SKILL.md and identify any subagents referenced via:
- `context: fork` with `agent` field
- Task tool invocations in the body (subagent_type references)
- References to `.claude/agents/` files

### Step 2: Parse Skill Structure

Extract from frontmatter (see [references/frontmatter-mapping.md](references/frontmatter-mapping.md)):

| Claude Code Field | Kilocode Equivalent |
|------------------|---------------------|
| `name` | `name` (must match directory) |
| `description` | `description` |
| `argument-hint` | Not supported (document in description) |
| `disable-model-invocation` | Not supported (skills auto-trigger) |
| `user-invocable` | Not supported |
| `allowed-tools` | Not supported |
| `model` | Not supported |
| `context` | Not supported |
| `agent` | Convert to mode reference |
| `hooks` | Not supported |

Extract from body:
- Main instructions → preserve as Kilocode instructions
- `$ARGUMENTS` → note for manual invocation
- `!`command`` dynamic context → convert to static or note as limitation
- Subagent references → convert to mode invocations

### Step 3: Identify Subagents to Convert

Scan for subagent usage patterns:

1. **`context: fork` skills**: The skill runs as a subagent
2. **Task tool calls**: `Task(subagent_type=X)` patterns
3. **Agent references**: `~/.claude/agents/{name}.md`

For each referenced agent/subagent:
- Use `/claude-agent-to-kilomode -a {agent}` to convert
- Update skill instructions to reference the new mode

### Step 4: Handle Mode-Specific Placement

Kilocode supports mode-specific skill directories:
- `skills/` - Available to all modes
- `skills-code/` - Code mode only
- `skills-architect/` - Architect mode only
- `skills-{mode-slug}/` - Custom mode only

**If `-a` flag:** Place in generic `skills/` directory
**If interactive:** Ask user about mode restrictions

### Step 5: Generate Kilocode Skill (depends on `-a` flag)

**If `-a` flag (Auto Mode):**
- Convert immediately using extracted data
- Place in `~/.kilocode/skills/{name}/SKILL.md`
- Convert referenced subagents to modes automatically
- Output complete YAML + instructions

**If NO `-a` flag (Interactive Mode):**
Ask about unclear aspects:
1. Mode restrictions? (all modes, code only, architect only, custom mode)
2. How to handle `$ARGUMENTS`? (manual invocation guide in description)
3. How to handle dynamic context `!`command``? (static snapshot vs remove)
4. Convert referenced subagents to modes?
5. Where to save? (global `~/.kilocode/skills/` or project `.kilocode/skills/`)

## Output Format

```yaml
---
name: skill-name
description: What this skill does and when to use it
license: MIT
---

# Skill Name

Instructions converted from Claude Code skill.

## Usage

[Converted workflow instructions]

## Notes

- Original used $ARGUMENTS for: [usage description]
- Invoke manually with: "use the skill-name skill to [action]"
```

## Conversion Examples

### Input (Claude Code Skill)
```yaml
---
name: commit
description: Quick commit and push with minimal messages
model: haiku
allowed-tools: Bash(git :*)
---

# Commit

Quick commit with conventional message format.

## Context

- Git state: !`git status`
- Recent commits: !`git log --oneline -5`

## Workflow

1. Analyze: Review git status
2. Generate commit message
3. Commit: `git commit -m "message"`
4. Push: `git push`
```

### Output (Kilocode Skill)
```yaml
---
name: commit
description: Quick commit and push with minimal, clean messages. Use when you want to commit current changes with auto-generated conventional commit message.
---

# Commit

Quick commit with conventional message format, then push.

## Workflow

1. **Analyze**: Run `git status` to review changes
2. **Generate commit message**:
   - Format: `type(scope): brief description`
   - Types: feat, fix, update, docs, chore, refactor, test, perf, revert
   - Under 72 chars, imperative mood, lowercase after colon
3. **Commit**: `git commit -m "message"`
4. **Push**: `git push`

## Rules

- NO INTERACTION: Never ask questions - analyze and commit
- AUTO-STAGE: If nothing staged, stage everything
- AUTO-PUSH: Always push after committing

## Notes

- This skill was converted from Claude Code
- Original used dynamic context injection (!`command`) for git state
- Kilocode will need to run git commands explicitly
```

### Workflow Skill Example

**Input (Claude Code Workflow Skill with Subagents):**
```yaml
---
name: apex
description: APEX methodology implementation workflow
argument-hint: "[-a] <task description>"
---

## Workflow
1. Load step-01-analyze.md → gather context
2. Load step-02-plan.md → create strategy

### Agent Usage
- Task(subagent_type=explore-codebase) for finding patterns
- Task(subagent_type=explore-docs) for documentation
```

**Output (Kilocode Skill + Modes):**

Creates skill AND converts subagents to modes:
1. `skills/apex/SKILL.md` - Main skill
2. `.kilocodemodes` - New modes for explore-codebase, explore-docs

```yaml
# In .kilocodemodes
customModes:
  - slug: explore-codebase
    name: "🔍 Codebase Explorer"
    description: Find existing patterns and files
    roleDefinition: |
      You are a codebase explorer...
    groups:
      - read
```

## Limitations to Document

When converting, document these limitations for users:

1. **No dynamic context injection**: `!`command`` syntax doesn't exist in Kilocode
2. **No argument passing**: $ARGUMENTS substitution not supported
3. **No tool restrictions**: `allowed-tools` not configurable per skill
4. **No model selection**: `model` field not available
5. **No subagent forks**: `context: fork` requires mode conversion
6. **Skills auto-trigger**: No `disable-model-invocation` equivalent

## Reference

See [references/frontmatter-mapping.md](references/frontmatter-mapping.md) for complete field mapping.
See [references/conversion-patterns.md](references/conversion-patterns.md) for common conversion patterns.
