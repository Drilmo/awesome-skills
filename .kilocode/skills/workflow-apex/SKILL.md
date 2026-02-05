---
name: workflow-apex
description: Systematic implementation using APEX methodology (Analyze-Plan-Execute-eXamine) with parallel subtasks, self-validation, and optional adversarial review. Use when implementing features, fixing bugs, or making code changes that benefit from structured workflow.
---

<objective>
Execute systematic implementation workflows using the APEX methodology. This skill uses progressive step loading to minimize context usage and supports saving outputs for review and resumption.
</objective>

<prerequisites>
## Prerequisites

This skill orchestrates multiple subtasks and must be run in **orchestrator** mode.

**Before starting:** Use `switch_mode` to switch to orchestrator mode.

**Required modes:** Ensure these modes are configured in `.kilocodemodes`:
- `explore-codebase` - For finding patterns
- `explore-docs` - For researching documentation
- `websearch` - For web research
- `code-reviewer` - For adversarial review
</prerequisites>

<quick_start>
**Basic usage:**

```
use the workflow-apex skill to add authentication middleware
```

**Recommended workflow (autonomous with save):**

```
use the workflow-apex skill with -a -s to implement user registration
```

**With adversarial review:**

```
use the workflow-apex skill with -a -x -s to fix login bug
```

**Flags:**

- `-a` (auto): Skip confirmations
- `-s` (save): Save outputs to `.claude/output/apex/`
- `-x` (examine): Include adversarial code review
- `-t` (test): Create and run tests
- `-pr` (pull-request): Create PR at end

See `<parameters>` for complete flag list.
</quick_start>

<parameters>

<flags>
**Enable flags (turn ON):**
| Short | Long | Description |
|-------|------|-------------|
| `-a` | `--auto` | Autonomous mode: skip confirmations, auto-approve plans |
| `-x` | `--examine` | Auto-examine mode: proceed to adversarial review |
| `-s` | `--save` | Save mode: output each step to `.claude/output/apex/` |
| `-t` | `--test` | Test mode: include test creation and runner steps |
| `-e` | `--economy` | Economy mode: no subtasks, save tokens (for limited plans) |
| `-r` | `--resume` | Resume mode: continue from a previous task |
| `-b` | `--branch` | Branch mode: verify not on main, create branch if needed |
| `-pr` | `--pull-request` | PR mode: create pull request at end (enables -b) |
| `-i` | `--interactive` | Interactive mode: configure flags via questions |

**Disable flags (turn OFF):**
| Short | Long | Description |
|-------|------|-------------|
| `-A` | `--no-auto` | Disable auto mode |
| `-X` | `--no-examine` | Disable examine mode |
| `-S` | `--no-save` | Disable save mode |
| `-T` | `--no-test` | Disable test mode |
| `-E` | `--no-economy` | Disable economy mode |
| `-B` | `--no-branch` | Disable branch mode |
| `-PR` | `--no-pull-request` | Disable PR mode |
</flags>

<examples>
```bash
# Basic
use apex skill to add auth middleware

# Autonomous (skip confirmations)
use apex skill with -a to add auth middleware

# Save outputs + examine
use apex skill with -a -x -s to add auth middleware

# Full workflow with tests
use apex skill with -a -x -s -t to add auth middleware

# With PR creation
use apex skill with -a -pr to add auth middleware

# Resume previous task
use apex skill with -r 01-auth-middleware

# Economy mode (save tokens)
use apex skill with -e to add auth middleware

# Interactive flag config
use apex skill with -i to add auth middleware
```
</examples>

<parsing_rules>
**Flag parsing:**

1. Defaults loaded from `steps/step-00-init.md` `<defaults>` section
2. Command-line flags override defaults (enable with lowercase `-x`, disable with uppercase `-X`)
3. Flags removed from input, remainder becomes `{task_description}`
4. Task ID generated as `NN-kebab-case-description`

For detailed parsing algorithm, see `steps/step-00-init.md`.
</parsing_rules>

</parameters>

<output_structure>
**When `{save_mode}` = true:**

All outputs saved to PROJECT directory:
```
.claude/output/apex/{task-id}/
├── 00-context.md # Params, user request, timestamp
├── 01-analyze.md # Analysis findings
├── 02-plan.md # Implementation plan
├── 03-execute.md # Execution log
├── 04-validate.md # Validation results
├── 05-examine.md # Review findings (if -x)
├── 06-resolve.md # Resolution log (if -x)
├── 07-tests.md # Test analysis and creation (if --test)
├── 08-run-tests.md # Test runner log (if --test)
└── 09-finish.md # Workflow finish and PR creation (if --pull-request)
```

**00-context.md structure:**
```markdown
# APEX Task: {task_id}

**Created:** {timestamp}
**Task:** {task_description}

## Flags
- Auto mode: {auto_mode}
- Examine mode: {examine_mode}
- Save mode: {save_mode}
- Test mode: {test_mode}

## User Request
{original user input}

## Acceptance Criteria
- [ ] AC1: {inferred criterion}
- [ ] AC2: {inferred criterion}
```

</output_structure>

<resume_workflow>
**Resume mode (`-r {task-id}`):**

When provided, step-00 will:

1. Locate the task folder in `.claude/output/apex/`
2. Restore state from `00-context.md`
3. Find the last completed step
4. Continue from the next step

Supports partial matching (e.g., `-r 01` finds `01-add-auth-middleware`).

For implementation details, see `steps/step-00-init.md`.
</resume_workflow>

<workflow>
**Standard flow:**
1. Parse flags and task description
2. If `-r`: Execute resume workflow
3. If `-s`: Create output folder and 00-context.md
4. Load step-01-analyze.md → gather context
5. Load step-02-plan.md → create strategy
6. Load step-03-execute.md → implement
7. Load step-04-validate.md → verify
8. If `--test`: Load step-07-tests.md → analyze and create tests
9. If `--test`: Load step-08-run-tests.md → run until green
10. If `-x` or user requests: Load step-05-examine.md → adversarial review
11. If findings: Load step-06-resolve.md → fix findings
12. If `-pr`: Load step-09-finish.md → create pull request
</workflow>

<state_variables>
**Persist throughout all steps:**

| Variable                | Type    | Description                                            |
| ----------------------- | ------- | ------------------------------------------------------ |
| `{task_description}`    | string  | What to implement (flags removed)                      |
| `{feature_name}`        | string  | Kebab-case name without number (e.g., `add-auth-middleware`) |
| `{task_id}`             | string  | Full identifier with number (e.g., `01-add-auth-middleware`) |
| `{acceptance_criteria}` | list    | Success criteria (inferred or explicit)                |
| `{auto_mode}`           | boolean | Skip confirmations, use recommended options            |
| `{examine_mode}`        | boolean | Auto-proceed to adversarial review                     |
| `{save_mode}`           | boolean | Save outputs to .claude/output/apex/                   |
| `{test_mode}`           | boolean | Include test steps (07-08)                             |
| `{economy_mode}`        | boolean | No subtasks, direct tool usage only                    |
| `{branch_mode}`         | boolean | Verify not on main, create branch if needed            |
| `{pr_mode}`             | boolean | Create pull request at end                             |
| `{interactive_mode}`    | boolean | Configure flags interactively                          |
| `{resume_task}`         | string  | Task ID to resume (if -r provided)                     |
| `{output_dir}`          | string  | Full path to output directory                          |
| `{branch_name}`         | string  | Created branch name (if branch_mode)                   |

</state_variables>

<entry_point>

**FIRST ACTION:** Load `steps/step-00-init.md`

Step 00 handles:

- Flag parsing (-a, -x, -s, -r, --test)
- Resume mode detection and task lookup
- Output folder creation (if save_mode)
- 00-context.md creation (if save_mode)
- State variable initialization

After initialization, step-00 loads step-01-analyze.md.

</entry_point>

<step_files>
**Progressive loading - only load current step:**

| Step | File                         | Purpose                                              |
| ---- | ---------------------------- | ---------------------------------------------------- |
| 00   | `steps/step-00-init.md`      | Parse flags, create output folder, initialize state  |
| 01   | `steps/step-01-analyze.md`   | Smart context gathering with 1-10 parallel subtasks based on complexity |
| 02   | `steps/step-02-plan.md`      | File-by-file implementation strategy                 |
| 03   | `steps/step-03-execute.md`   | Todo-driven implementation                           |
| 04   | `steps/step-04-validate.md`  | Self-check and validation                            |
| 05   | `steps/step-05-examine.md`   | Adversarial code review (optional)                   |
| 06   | `steps/step-06-resolve.md`   | Finding resolution (optional)                        |
| 07   | `steps/step-07-tests.md`     | Test analysis and creation (if --test)               |
| 08   | `steps/step-08-run-tests.md` | Test runner loop until green (if --test)             |
| 09   | `steps/step-09-finish.md`    | Create pull request (if --pull-request)              |

</step_files>

<execution_rules>

- **Load one step at a time** - Only load the current step file
- **ULTRA THINK** before major decisions
- **Persist state variables** across all steps
- **Follow next_step directive** at end of each step
- **Save outputs** if `{save_mode}` = true (append to step file)
- **Use parallel subtasks** for independent exploration tasks (via `new_task`)

## 🧠 Smart Subtask Strategy in Analyze Phase

The analyze phase (step-01) uses **adaptive subtask launching** (unless economy_mode):

**Available modes for subtasks:**
- `explore-codebase` - Find existing patterns, files, utilities
- `explore-docs` - Research library docs (use when unfamiliar with API)
- `websearch` - Find approaches, best practices, gotchas

**Launch 1-10 subtasks based on task complexity:**

| Complexity | Subtasks | When |
|------------|----------|------|
| Simple | 1-2 | Bug fix, small tweak |
| Medium | 2-4 | New feature in familiar stack |
| Complex | 4-7 | Unfamiliar libraries, integrations |
| Major | 6-10 | Multiple systems, many unknowns |

**BE SMART:** Analyze what you actually need before launching. Don't over-launch for simple tasks, don't under-launch for complex ones.

</execution_rules>

<save_output_pattern>
**When `{save_mode}` = true:**

Step-00 runs `scripts/setup-templates.sh` to initialize all output files from `templates/` directory.

**Each step then:**

1. Run `scripts/update-progress.sh {task_id} {step_num} {step_name} "in_progress"`
2. Append findings/outputs to the pre-created step file
3. Run `scripts/update-progress.sh {task_id} {step_num} {step_name} "complete"`

**Template system benefits:**

- Reduces token usage by ~75% (1,350 tokens saved per workflow)
- Templates in `templates/` directory (not inline in steps)
- Scripts handle progress tracking automatically
- See `templates/README.md` for details

</save_output_pattern>

<success_criteria>

- Each step loaded progressively
- All validation checks passing
- Outputs saved if `{save_mode}` enabled
- Tests passing if `{test_mode}` enabled
- Clear completion summary provided
</success_criteria>

<conversion_notes>
## Conversion Notes

This skill was converted from Claude Code. Key differences:

1. **Subtask mechanism**: Uses `new_task` to create subtasks in specific modes
   - Original: `Task(subagent_type=explore-codebase)`
   - Converted: Use `new_task` in **explore-codebase** mode

2. **Orchestrator requirement**: This skill MUST run in **orchestrator** mode
   to spawn subtasks

3. **Parallel subtasks**: Create multiple `new_task` calls for parallel execution

4. **Mode dependencies**: Requires these modes in `.kilocodemodes`:
   - explore-codebase
   - explore-docs
   - websearch
   - code-reviewer

5. **Skill invocation**: Original `/apex` command becomes manual invocation:
   "use the workflow-apex skill to [task]"
</conversion_notes>
