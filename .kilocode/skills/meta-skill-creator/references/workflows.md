# Workflow Patterns

## Sequential Workflows

For complex tasks, break operations into clear, sequential steps. It is often helpful to give the AI an overview of the process towards the beginning of SKILL.md:

```markdown
Filling a PDF form involves these steps:

1. Analyze the form (run analyze_form.py)
2. Create field mapping (edit fields.json)
3. Validate mapping (run validate_fields.py)
4. Fill the form (run fill_form.py)
5. Verify output (run verify_output.py)
```

## Conditional Workflows

For tasks with branching logic, guide the AI through decision points:

```markdown
1. Determine the modification type:
   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow: [steps]
3. Editing workflow: [steps]
```

## Subtask Patterns (Kilocode-specific)

For skills that need to spawn parallel or isolated work:

### Parallel Research Pattern

```markdown
## Context Gathering

Launch parallel research subtasks:
1. Use `new_task` in **explore-codebase** mode to find existing patterns
2. Use `new_task` in **explore-docs** mode to research library documentation
3. Use `new_task` in **websearch** mode to find best practices

Wait for all subtasks to complete, then synthesize findings.
```

### Sequential Mode Transitions

```markdown
## Implementation Flow

1. Use `switch_mode` to **architect** mode for design
2. Create implementation plan
3. Use `switch_mode` to **code** mode for implementation
4. Implement the changes
5. Use `switch_mode` to **code-reviewer** mode for validation
```

### Isolated Subtask Pattern

```markdown
## Code Review

Create an isolated review subtask:
- Use `new_task` in **code-reviewer** mode
- Provide the files to review
- Wait for review findings
- Address any issues found
```
