# Frontmatter Field Mapping

Complete mapping of Claude Code skill frontmatter to Kilocode format.

## Supported Fields

| Claude Code | Kilocode | Status | Notes |
|-------------|----------|--------|-------|
| `name` | `name` | ✅ Supported | Must match directory name |
| `description` | `description` | ✅ Supported | Max 1024 chars |

## Unsupported Fields

| Field | Kilocode | Conversion Strategy |
|-------|----------|---------------------|
| `argument-hint` | ❌ | Add usage pattern to description |
| `disable-model-invocation` | ❌ | Note in description if manual-only |
| `user-invocable` | ❌ | Skills always auto-trigger |
| `allowed-tools` | ❌ | Document as requirements in body |
| `model` | ❌ | Note in comments |
| `context` | ❌ | Convert to mode if `fork` |
| `agent` | ❌ | Reference converted mode |
| `hooks` | ❌ | Convert to manual steps |

## Kilocode-Only Fields

| Field | Description | Default |
|-------|-------------|---------|
| `license` | License identifier | Omit |
| `compatibility` | Version info | Omit |
| `metadata` | Custom metadata | Omit |

## Conversion Patterns

### argument-hint → description

```yaml
# Claude Code
argument-hint: "[filename] [format]"
description: Convert file to format

# Kilocode
description: Convert file to format. Usage - "use convert skill on [filename] to [format]"
```

### disable-model-invocation → description

```yaml
# Claude Code
disable-model-invocation: true
description: Deploy to production

# Kilocode
description: Deploy to production. Manual invocation only - "use the deploy skill"
```

### allowed-tools → body

```yaml
# Claude Code
allowed-tools: Read, Grep, Glob

# Kilocode body
## Requirements
This skill uses read-only operations:
- Read files
- Search with grep
- Find files with glob

Do NOT modify files during this workflow.
```

### model → comment

```yaml
# Claude Code
model: haiku

# Kilocode body (as comment or note)
## Notes
- Designed for quick operations (original used haiku model)
```

### context: fork → mode reference

```yaml
# Claude Code
context: fork
agent: Explore

# Kilocode body
This workflow should be executed in **Codebase Explorer** mode.
Ensure the explore-codebase mode is configured in .kilocodemodes
```

### hooks → manual steps

```yaml
# Claude Code
hooks:
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "npm run lint"

# Kilocode body
## After Editing
After making any file edits, run:
```bash
npm run lint
```
This ensures code quality is maintained.
```

## Description Enhancement

Kilocode relies on description for auto-triggering. Enhance descriptions:

### Add Trigger Phrases

```yaml
# Claude Code
description: Fix spelling errors

# Kilocode
description: Fix spelling errors in files. Use when asked to fix typos, correct spelling, or proofread content.
```

### Include Usage Pattern

```yaml
# Claude Code
description: Create component
argument-hint: "<name>"

# Kilocode
description: Create React component. Use when asked to create component, scaffold component, or add new component. Specify name in request.
```

### Note Manual Invocation

```yaml
# Claude Code
description: Deploy app
disable-model-invocation: true

# Kilocode
description: Deploy application to production. Invoke manually with "use the deploy skill" - does not auto-trigger.
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
hooks:
  PostToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "echo 'Command executed'"
---

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
description: Review a pull request for code quality and issues. Manual invocation only - "use the review-pr skill on PR #123". Use when asked to review PR, check pull request, or analyze PR changes.
---

# PR Review

Review the specified pull request.

## Requirements

This skill requires:
- GitHub CLI (`gh`) installed and authenticated
- Read access to the repository

## Context Gathering

First, gather PR information:
1. Run `gh pr view [PR-NUMBER]` to see PR details
2. Run `gh pr diff [PR-NUMBER]` to see changes

## Process

1. **Analyze Changes**
   - Review the diff
   - Understand scope of changes

2. **Check for Issues**
   - Code quality problems
   - Security concerns
   - Performance impacts
   - Missing tests

3. **Provide Feedback**
   - Critical issues (must fix)
   - Warnings (should fix)
   - Suggestions (nice to have)

## After Running Commands

Log when commands are executed for debugging.

## Notes

- This skill was converted from Claude Code
- Original used dynamic context injection for PR info
- Kilocode executes gh commands explicitly
- Manual invocation: "use the review-pr skill on PR #[number]"
```
