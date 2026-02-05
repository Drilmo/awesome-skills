# Common Conversion Patterns

Patterns for converting different types of Claude Code skills to Kilocode format.

## Pattern 1: Simple Utility Skill

Skills with no special features - straightforward conversion.

### Claude Code
```yaml
---
name: fix-typos
description: Fix spelling and grammar errors in files
---

# Fix Typos

Scan files for spelling and grammar issues and fix them.

## Process
1. Read the file
2. Identify errors
3. Apply fixes
4. Report changes
```

### Kilocode
```yaml
---
name: fix-typos
description: Fix spelling and grammar errors in files. Use when asked to proofread, fix typos, or correct grammar.
---

# Fix Typos

Scan files for spelling and grammar issues and fix them.

## Process
1. Read the file
2. Identify errors
3. Apply fixes
4. Report changes
```

**Changes**: Just enhanced description for better auto-triggering.

---

## Pattern 2: Skill with Dynamic Context

Skills using `!`command`` for context injection.

### Claude Code
```yaml
---
name: commit
description: Quick commit
---

# Commit

## Context
- Status: !`git status`
- Diff: !`git diff --cached --stat`
- Recent: !`git log --oneline -5`

## Workflow
1. Generate message
2. Commit
3. Push
```

### Kilocode
```yaml
---
name: commit
description: Quick commit and push with auto-generated message. Use when asked to commit, save changes, or push code.
---

# Commit

## Workflow

1. **Gather Context**
   - Run `git status` to see changes
   - Run `git diff --cached --stat` to see staged changes
   - Run `git log --oneline -5` to see recent commit style

2. **Generate Message**
   - Format: `type(scope): description`
   - Match existing commit style

3. **Commit and Push**
   - `git commit -m "message"`
   - `git push`
```

**Changes**: Converted dynamic context to explicit workflow steps.

---

## Pattern 3: Skill with Arguments

Skills using `$ARGUMENTS` for user input.

### Claude Code
```yaml
---
name: create-component
description: Create a React component
argument-hint: "<component-name>"
---

Create component `$ARGUMENTS` in src/components/
```

### Kilocode
```yaml
---
name: create-component
description: Create a React component. Use when asked to create component, add new component, or scaffold component. Specify the component name in your request.
---

# Create Component

Create the specified component in `src/components/`.

## Usage

Tell the AI: "use the create-component skill to create [ComponentName]"

## Process

1. Create file at `src/components/[ComponentName]/index.tsx`
2. Add basic component structure
3. Export from barrel file if exists
```

**Changes**: Document argument passing in description and body.

---

## Pattern 4: Workflow Skill with Steps

Multi-step skills with progressive loading.

### Claude Code
```yaml
---
name: apex
description: APEX implementation workflow
argument-hint: "[-a] <task>"
---

<workflow>
1. Load step-01-analyze.md
2. Load step-02-plan.md
3. Load step-03-execute.md
</workflow>
```

### Kilocode
```yaml
---
name: apex
description: APEX implementation workflow. Use for systematic feature implementation with analyze-plan-execute phases.
---

# APEX Workflow

Systematic implementation using Analyze-Plan-Execute methodology.

## Phase 1: Analyze

Gather context about the codebase:
- Find related files
- Understand existing patterns
- Document dependencies

## Phase 2: Plan

Create implementation strategy:
- List files to modify
- Define change sequence
- Identify risks

## Phase 3: Execute

Implement changes:
- Follow the plan
- Make incremental changes
- Validate each step

## Phase 4: Verify

Confirm implementation:
- Run tests
- Check for regressions
- Review changes
```

**Changes**: Inline the step content or reference separate files within skill directory.

---

## Pattern 5: Skill with Subagent Calls

Skills that use Task tool to delegate to subagents.

### Claude Code
```yaml
---
name: refactor
description: Refactor code across codebase
---

## Process
1. Find files with Grep
2. Launch parallel Snipper agents:
   - Task(subagent_type=Snipper) for batch 1
   - Task(subagent_type=Snipper) for batch 2
3. Validate with lint
```

### Kilocode

First, convert `Snipper` agent to mode (use `/claude-agent-to-kilomode -a snipper`):

```yaml
# In .kilocodemodes
customModes:
  - slug: snipper
    name: "⚡ Snipper"
    description: Fast code modifications
    roleDefinition: |
      You are a rapid code modification specialist.
    groups:
      - read
      - edit
```

Then update skill:

```yaml
---
name: refactor
description: Refactor code across codebase using parallel modifications. Use when renaming, updating patterns, or applying consistent changes.
---

# Refactor

## Process

1. **Discovery**: Find all files matching the pattern
   - Use grep/glob to locate occurrences

2. **Group Files**: Batch files for processing

3. **Apply Changes**: For each batch:
   - Switch to Snipper mode for fast modifications
   - Apply the refactor pattern
   - Return to default mode

4. **Validate**: Run lint and type checks

## Notes

- This skill works with the Snipper mode for fast modifications
- Ensure Snipper mode is configured in .kilocodemodes
```

---

## Pattern 6: Skill with Tool Restrictions

Skills with `allowed-tools` field.

### Claude Code
```yaml
---
name: safe-explore
description: Read-only codebase exploration
allowed-tools: Read, Grep, Glob
---

Explore the codebase without making changes.
```

### Kilocode

Option A: Convert to Mode (recommended for strict enforcement)

```yaml
# In .kilocodemodes
customModes:
  - slug: safe-explore
    name: "🔍 Safe Explorer"
    description: Read-only codebase exploration
    roleDefinition: |
      You are a read-only codebase explorer.
      NEVER modify any files.
    groups:
      - read  # Only read group, no edit/command
```

Option B: Keep as Skill with documented restrictions

```yaml
---
name: safe-explore
description: Read-only codebase exploration. Use when you need to analyze code without making changes.
---

# Safe Explore

Explore the codebase without making any modifications.

## IMPORTANT: Read-Only Mode

This skill is for exploration only:
- DO NOT edit any files
- DO NOT write new files
- DO NOT run commands that modify state

## Allowed Operations

- Read files
- Search with grep patterns
- Find files with glob patterns
- Analyze and summarize findings
```

---

## Pattern 7: Skill with `context: fork`

Skills that run in isolated context.

### Claude Code
```yaml
---
name: deep-research
description: Thorough research in isolated context
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly and return findings.
```

### Kilocode

Convert to Mode instead (fork = isolated context = mode):

```yaml
# In .kilocodemodes
customModes:
  - slug: deep-research
    name: "🔬 Deep Research"
    description: Thorough isolated research mode
    roleDefinition: |
      You are a research specialist working in isolation.
      Focus entirely on gathering information.
      Do not make code changes.
    groups:
      - read
      - browser
    customInstructions: |
      Research the topic thoroughly:
      1. Find relevant files
      2. Analyze patterns
      3. Summarize findings with references
```

Then reference from other skills:

```yaml
---
name: analyze
description: Analyze codebase for implementation
---

For deep research tasks, switch to the deep-research mode.
```

---

## Subagent to Mode Reference Table

Common Claude Code subagents and their Kilocode mode equivalents:

| Claude Code Subagent | Kilocode Mode | Groups |
|---------------------|---------------|--------|
| `Explore` | 🔍 Codebase Explorer | read |
| `Plan` | 📋 Architect | read |
| `Snipper` | ⚡ Snipper | read, edit |
| `code-reviewer` | 🔎 Code Reviewer | read |
| `general-purpose` | 💻 Code (default) | read, edit, command |
| `test-runner` | 🧪 Test Runner | read, edit, command |

---

## Handling Unsupported Features

### Feature: Hooks

Claude Code hooks have no Kilocode equivalent.

**Strategy**: Document as manual steps or remove

```yaml
# Claude Code
hooks:
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "npm run lint"

# Kilocode
## After Making Changes
Always run `npm run lint` after editing files.
```

### Feature: Model Selection

No per-skill model selection in Kilocode.

**Strategy**: Document performance expectations

```yaml
# Claude Code
model: haiku  # Fast model

# Kilocode - add note
## Notes
This skill is designed for quick operations.
```

### Feature: Session ID

`${CLAUDE_SESSION_ID}` not available in Kilocode.

**Strategy**: Use timestamps or generate IDs differently

```yaml
# Claude Code
Log to logs/${CLAUDE_SESSION_ID}.log

# Kilocode
Log to logs/[timestamp]-session.log
Generate timestamp with: date +%Y%m%d-%H%M%S
```
