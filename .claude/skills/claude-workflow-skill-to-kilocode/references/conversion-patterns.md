# Conversion Patterns

Common patterns for converting different types of Claude Code skills to Kilocode.

## Pattern 1: Simple Utility Skill

Skills with basic frontmatter and instructions.

### Claude Code
```yaml
---
name: fix-typos
description: Fix spelling and grammar errors in files
---

# Fix Typos

Scan files for spelling and grammar issues and fix them.
```

### Kilocode
```yaml
---
name: fix-typos
description: Fix spelling and grammar errors in files. Use when asked to proofread, fix typos, correct spelling, or fix grammar.
---

# Fix Typos

Scan files for spelling and grammar issues and fix them.
```

**Changes**: Enhanced description only.

---

## Pattern 2: Skill with Arguments

Skills using `$ARGUMENTS` substitution.

### Claude Code
```yaml
---
name: create-component
argument-hint: "<component-name>"
---

Create component `$ARGUMENTS` in src/components/
Follow existing patterns.
```

### Kilocode
```yaml
---
name: create-component
description: Create a React component. Use when asked to create, scaffold, or add a component. Specify component name in your request.
---

# Create Component

Create the specified component in `src/components/`.

## Usage

Tell the AI: "use the create-component skill to create MyComponent"

## Process

1. Determine component name from request
2. Create file at `src/components/{ComponentName}/index.tsx`
3. Follow existing component patterns
4. Export from barrel file if exists
```

---

## Pattern 3: Dynamic Context Injection

Skills using `!`command`` syntax.

### Claude Code
```yaml
---
name: commit
description: Quick commit
---

## Context
- Status: !`git status`
- Branch: !`git branch --show-current`
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
   Run these commands to understand current state:
   - `git status` - see changes
   - `git branch --show-current` - identify branch
   - `git log --oneline -5` - see recent commit style

2. **Generate Message**
   Create conventional commit message:
   - Format: `type(scope): description`
   - Match existing style from recent commits

3. **Commit and Push**
   ```bash
   git add .
   git commit -m "message"
   git push
   ```
```

---

## Pattern 4: Tool Restrictions

Skills with `allowed-tools` field.

### Claude Code
```yaml
---
name: safe-explore
description: Read-only exploration
allowed-tools: Read, Grep, Glob
---

Explore the codebase without making changes.
```

### Kilocode (Option A: Mode)
```yaml
# .kilocodemodes
customModes:
  - slug: safe-explore
    name: "🔍 Safe Explorer"
    description: Read-only codebase exploration
    roleDefinition: |
      You are a read-only explorer.
      NEVER modify any files.
    groups:
      - read
```

### Kilocode (Option B: Skill with notes)
```yaml
---
name: safe-explore
description: Read-only codebase exploration. Use when analyzing code without changes.
---

# Safe Explore

Explore the codebase without making modifications.

## Important: Read-Only Mode

This skill is for exploration only:
- ✅ Read files
- ✅ Search with grep
- ✅ Find with glob
- ❌ DO NOT edit files
- ❌ DO NOT write files
- ❌ DO NOT run destructive commands
```

---

## Pattern 5: Subagent Execution

Skills with `context: fork` and `agent`, or using `Task` tool.

**Important Kilocode tools:**
- `switch_mode` - Change mode in current conversation
- `new_task` - Create a subtask (equivalent to Claude Code's Task tool)

### Claude Code
```yaml
---
name: deep-research
description: Research in isolated context
context: fork
agent: Explore
---

Research $ARGUMENTS thoroughly.
Find patterns and summarize.
```

### Kilocode

First, create mode:
```yaml
# .kilocodemodes
customModes:
  - slug: deep-research
    name: "🔬 Deep Research"
    description: Thorough isolated research
    roleDefinition: |
      You are a research specialist.
      Focus on gathering information.
      Do not make code changes.
    groups:
      - read
      - browser
```

Then, skill uses `new_task`:
```yaml
---
name: deep-research
description: Research a topic thoroughly. Use when deep analysis is needed. Requires deep-research mode.
---

# Deep Research

## Setup

Ensure the `deep-research` mode is configured in `.kilocodemodes`.

## Process

Use `new_task` to create a subtask in **deep-research** mode:

1. The subtask will research the specified topic:
   - Find relevant files
   - Analyze patterns
   - Search web if needed
2. Summarize findings with references

Note: `new_task` creates an isolated context similar to Claude Code's `context: fork`.
```

---

## Pattern 5b: Parallel Subagents

Skills that launch multiple agents in parallel.

### Claude Code
```markdown
Launch ALL agents in a SINGLE message:

**Agent 1:** Task(subagent_type=explore-codebase)
Find files related to the task.

**Agent 2:** Task(subagent_type=websearch)
Search for best practices.

**Agent 3:** Task(subagent_type=code-reviewer)
Review the existing implementation.
```

### Kilocode
```markdown
Create parallel subtasks using `new_task`:

1. **Subtask 1:** Use `new_task` in **explore-codebase** mode
   Find files related to the task.

2. **Subtask 2:** Use `new_task` in **websearch** mode
   Search for best practices.

3. **Subtask 3:** Use `new_task` in **code-reviewer** mode
   Review the existing implementation.

Note: Each `new_task` creates an independent subtask with its own context.
Wait for all subtasks to complete, then synthesize results.
```

---

## Pattern 6: Workflow with Steps

Multi-step skills with `steps/` directory.

### Claude Code Structure
```
workflow-skill/
├── SKILL.md
└── steps/
    ├── step-01-analyze.md
    ├── step-02-plan.md
    └── step-03-execute.md
```

### Kilocode (Option A: Inline)

Combine all steps into single SKILL.md:
```yaml
---
name: workflow-skill
description: Multi-phase workflow. Use for structured implementation.
---

# Workflow Skill

## Phase 1: Analyze

{content from step-01-analyze.md}

## Phase 2: Plan

{content from step-02-plan.md}

## Phase 3: Execute

{content from step-03-execute.md}
```

### Kilocode (Option B: Preserve Structure)

Keep steps directory:
```
workflow-skill/
├── SKILL.md
└── steps/
    ├── step-01-analyze.md (converted)
    ├── step-02-plan.md (converted)
    └── step-03-execute.md (converted)
```

Each step file converted to remove Claude Code specifics.

---

## Pattern 7: Skill with Hooks

Skills using lifecycle hooks.

### Claude Code
```yaml
---
name: safe-edit
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "git stash"
  PostToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "npm run lint"
---

Edit files safely with auto-backup.
```

### Kilocode
```yaml
---
name: safe-edit
description: Edit files with safety checks. Use when making changes that need rollback protection.
---

# Safe Edit

Edit files with automatic safety measures.

## Before Making Changes

Always run before editing:
```bash
git stash
```
This creates a backup of current state.

## Making Changes

Proceed with file edits.

## After Making Changes

Always run after editing:
```bash
npm run lint
```
This ensures code quality.

## Rollback if Needed

If changes need to be reverted:
```bash
git stash pop
```

## Notes

- Original skill had automatic hooks for pre/post actions
- Kilocode requires manual execution of these steps
- Consider creating a script to automate the workflow
```

---

## Pattern 8: Model-Specific Skill

Skills using specific models.

### Claude Code
```yaml
---
name: quick-fix
model: haiku
description: Fast, simple fixes
---

Fix the issue quickly. Minimal changes.
```

### Kilocode
```yaml
---
name: quick-fix
description: Fast, simple fixes with minimal changes. Use for quick patches, typo fixes, or simple updates. Designed for speed.
---

# Quick Fix

Fix the issue quickly with minimal changes.

## Guidelines

- Make smallest possible change
- Don't refactor surrounding code
- Don't add features
- Focus on the immediate fix

## Notes

- This skill was designed for fast execution
- Original used haiku model for speed
- Keep responses concise
```

---

## Subagent to Mode Quick Reference

| Claude Code | Kilocode Mode | Groups |
|-------------|---------------|--------|
| `Explore` | explore-codebase | read |
| `Plan` | architect | read |
| `general-purpose` | full-stack | read, edit, command |
| `Snipper` | snipper | read, edit |
| `code-reviewer` | code-reviewer | read |
| `test-runner` | test-runner | read, command |

---

## Claude Code vs Kilocode Tool Mapping

| Claude Code | Kilocode | Purpose |
|-------------|----------|---------|
| `Task(subagent_type=X)` | `new_task` + mode | Create subtask with isolated context |
| `context: fork` | `new_task` | Isolated execution context |
| - | `switch_mode` | Change mode in current conversation |

### When to use `new_task` vs `switch_mode`

**Use `new_task` when:**
- Need isolated context (like Claude Code's `context: fork`)
- Running parallel subtasks (like multiple Task calls)
- Want independent execution without affecting main conversation

**Use `switch_mode` when:**
- Want to change AI behavior in current conversation
- Don't need isolated context
- Sequential workflow within same conversation

### CRITICAL: Skills with subtasks require orchestrator mode

**If a skill uses `new_task` to spawn subtasks, it MUST be run in orchestrator mode.**

The orchestrator mode has the capability to create and manage subtasks. Before starting such a skill, the user must first `switch_mode` to orchestrator.

Example skill preamble:
```markdown
## Prerequisites

This skill orchestrates multiple subtasks and must be run in **orchestrator** mode.

**Before starting:** Use `switch_mode` to switch to orchestrator mode.

## Workflow

1. (In orchestrator mode) Create subtask in **explore-codebase** mode
2. (In orchestrator mode) Create subtask in **code-reviewer** mode
3. Synthesize results from subtasks
```
