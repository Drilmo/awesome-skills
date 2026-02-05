# Claude Code to Kilocode Frontmatter Mapping

Complete field-by-field mapping for converting Claude Code skill frontmatter to Kilocode format.

## Supported Fields

| Claude Code | Kilocode | Notes |
|------------|----------|-------|
| `name` | `name` | Must match directory name exactly in Kilocode |
| `description` | `description` | Keep under 1024 chars in Kilocode |

## Unsupported Fields (Conversion Notes)

### `argument-hint`

**Claude Code**: Shows hint during autocomplete (e.g., `[issue-number]`)
**Kilocode**: Not supported

**Conversion Strategy**:
- Document expected arguments in the description
- Add usage examples in the skill body

Example:
```yaml
# Claude Code
argument-hint: "[filename] [format]"
description: Convert file to different format

# Kilocode
description: Convert file to different format. Usage - tell the AI "use convert-file skill on [filename] to [format]"
```

### `disable-model-invocation`

**Claude Code**: Prevents AI from auto-loading skill (user must invoke with `/name`)
**Kilocode**: Not supported - skills always auto-trigger based on description

**Conversion Strategy**:
- Make description very specific so it only triggers for exact matches
- Document that manual invocation is: "use the [skill-name] skill to..."

### `user-invocable`

**Claude Code**: `false` hides from `/` menu (AI-only skill)
**Kilocode**: Not supported

**Conversion Strategy**:
- Skills are always discoverable in Kilocode
- Document intended usage in description

### `allowed-tools`

**Claude Code**: Restricts tools the AI can use when skill is active
**Kilocode**: Not supported per-skill

**Conversion Strategy**:
- Document tool requirements in the skill body
- Consider creating a mode with `groups` restrictions instead

Example conversion:
```yaml
# Claude Code
allowed-tools: Read, Grep, Glob

# Kilocode - document in body
## Requirements
This skill requires read-only operations. Do NOT edit or write files.
```

### `model`

**Claude Code**: Sets AI model for skill execution (haiku, sonnet, opus)
**Kilocode**: Not supported per-skill

**Conversion Strategy**:
- Remove field
- For haiku skills (fast/cheap), document that it's designed for quick tasks

### `context: fork`

**Claude Code**: Runs skill in isolated subagent context
**Kilocode**: Not supported directly

**Conversion Strategy**:
- Convert the skill to a Kilocode mode instead
- Reference the mode from other skills using "switch to [mode-slug] mode"

### `agent`

**Claude Code**: Specifies subagent type when `context: fork` is used
**Kilocode**: Convert to mode reference

**Conversion Strategy**:
- Convert the referenced agent to a Kilocode mode using `/claude-agent-to-kilomode`
- Update skill to reference mode: "Use [mode-name] mode for this task"

### `hooks`

**Claude Code**: Lifecycle hooks (PreToolUse, PostToolUse, etc.)
**Kilocode**: Not supported

**Conversion Strategy**:
- Remove hooks
- Document any pre/post actions as manual steps
- Consider if Kilocode MCP could provide similar functionality

## Optional Kilocode Fields

Fields available in Kilocode but not in Claude Code:

### `license`

**Kilocode only**: Optional license declaration (e.g., `Apache-2.0`, `MIT`)
**Default**: Omit or use `MIT`

### `compatibility`

**Kilocode only**: Optional compatibility metadata
**Usage**: Rarely needed, omit unless distributing

### `metadata`

**Kilocode only**: Optional arbitrary metadata object
**Usage**: For custom integrations, usually omit

## Body Content Mapping

### Dynamic Context Injection

**Claude Code**: `!`command`` executes shell command and injects output
**Kilocode**: Not supported

**Conversion Strategy**:
```markdown
# Claude Code
## Context
- Git status: !`git status`
- Branch: !`git branch --show-current`

# Kilocode
## Context Gathering
First, gather context by running:
1. `git status` - to see current changes
2. `git branch --show-current` - to identify current branch
```

### String Substitutions

**Claude Code**: `$ARGUMENTS`, `$ARGUMENTS[N]`, `$N`, `${CLAUDE_SESSION_ID}`
**Kilocode**: Not supported

**Conversion Strategy**:
```markdown
# Claude Code
Process file $ARGUMENTS[0] with format $ARGUMENTS[1]

# Kilocode
Process the specified file with the specified format.
User should specify: "use this skill on [filename] with [format]"
```

## Complete Conversion Example

### Claude Code Skill
```yaml
---
name: review-pr
description: Review a pull request
argument-hint: "[pr-number]"
model: sonnet
allowed-tools: Bash(gh *), Read, Grep, Glob
disable-model-invocation: true
---

# PR Review

Review PR #$ARGUMENTS

## Context
- PR info: !`gh pr view $ARGUMENTS`
- PR diff: !`gh pr diff $ARGUMENTS`

## Process
1. Analyze changes
2. Check for issues
3. Provide feedback
```

### Kilocode Skill
```yaml
---
name: review-pr
description: Review a pull request. Use when asked to review PR, analyze pull request, or check PR changes. Invoke with "use the review-pr skill on PR #123"
---

# PR Review

Review the specified pull request.

## Context Gathering

First, gather PR information:
1. Run `gh pr view [PR-NUMBER]` to see PR details
2. Run `gh pr diff [PR-NUMBER]` to see changes

## Process

1. Analyze the diff for:
   - Code quality issues
   - Security concerns
   - Performance impacts
   - Test coverage

2. Check for common issues:
   - Missing error handling
   - Hardcoded values
   - Breaking changes

3. Provide structured feedback:
   - Critical issues (must fix)
   - Warnings (should fix)
   - Suggestions (nice to have)

## Notes

- This skill requires the GitHub CLI (`gh`) to be installed
- Original Claude Code skill used dynamic context injection
- Manual invocation: "use the review-pr skill on PR #[number]"
```
