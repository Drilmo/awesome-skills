---
name: step-03-execute
description: Todo-driven implementation - execute the plan file by file
prev_step: steps/step-02-plan.md
next_step: steps/step-04-validate.md
---

# Step 3: Execute (Implementation)

## MANDATORY EXECUTION RULES (READ FIRST):

- 🛑 NEVER deviate from the approved plan
- 🛑 NEVER add features not in the plan (scope creep)
- 🛑 NEVER modify files without reading them first
- ✅ ALWAYS follow the plan file-by-file
- ✅ ALWAYS mark todos complete immediately after each task
- ✅ ALWAYS read files BEFORE editing them
- 📋 YOU ARE AN IMPLEMENTER following a plan, not a designer
- 💬 FOCUS on executing the plan exactly as approved
- 🚫 FORBIDDEN to add "improvements" not in the plan

## EXECUTION PROTOCOLS:

- 🎯 Create todos from plan before starting
- 💾 Mark todos complete immediately after each task
- 📖 Read each file BEFORE modifying it
- 🚫 FORBIDDEN to have multiple todos in_progress simultaneously

## CONTEXT BOUNDARIES:

- Plan from step-02 is approved and must be followed
- Files to modify are known from the plan
- Patterns to follow are documented from step-01
- Don't add features - stick to the plan

## YOUR TASK:

Execute the approved implementation plan file-by-file, tracking progress with todos.

---

<available_state>
From previous steps:

| Variable | Description |
|----------|-------------|
| `{task_description}` | What to implement |
| `{task_id}` | Kebab-case identifier |
| `{auto_mode}` | Skip confirmations |
| `{save_mode}` | Save outputs to files |
| `{output_dir}` | Path to output (if save_mode) |
| Implementation plan | File-by-file changes from step-02 |
| Patterns | How to implement from step-01 |
</available_state>

---

## EXECUTION SEQUENCE:

### 1. Initialize Save Output (if save_mode)

**If `{save_mode}` = true:**

```bash
bash {skill_dir}/scripts/update-progress.sh "{task_id}" "03" "execute" "in_progress"
```

Append logs to `{output_dir}/03-execute.md` as you work.

### 2. Create Todos from Plan

Convert each file change from the plan into todos:

```
Plan entry:
#### `src/auth/handler.ts`
- Add `validateToken` function
- Handle error case: expired token

Becomes:
- [ ] src/auth/handler.ts: Add validateToken function
- [ ] src/auth/handler.ts: Handle expired token error
```

Use `update_todo_list` to create the full list.

### 3. Execute File by File

For each todo:

**3.1 Mark In Progress**
- Only ONE todo in_progress at a time

**3.2 Read Before Edit**
```
ALWAYS read the file before modifying:
- Understand current structure
- Find exact insertion points
- Verify patterns match expectations
```

**3.3 Implement Changes**
```
Make changes specified in the plan:
- Follow patterns from step-01 analysis
- Use exact names from plan
- Handle error cases as specified
- NO comments unless truly necessary
```

**3.4 Mark Complete Immediately**
- Mark todo complete RIGHT AFTER finishing
- Don't batch completions

**3.5 Log Progress (if save_mode)**
```markdown
### ✓ src/auth/handler.ts
- Added `validateToken` function (lines 45-78)
- Added error handling for expired tokens
**Timestamp:** {ISO}
```

### 4. Handle Blockers

**If `{auto_mode}` = true:**
→ Make reasonable decision and continue

**If `{auto_mode}` = false:**

Use `ask_followup_question`:
```
Encountered an issue. How should we proceed?

Options:
- Use alternative approach (Recommended): Description of alternative
- Skip this part: Continue without this change
- Stop for discussion: I want to discuss before continuing
```

### 5. Verify Implementation

After completing all todos:

```bash
pnpm run typecheck && pnpm run lint --fix
```

Fix any errors immediately.

### 6. Implementation Summary

```
**Implementation Complete**

**Files Modified:**
- `src/auth/handler.ts` - Added validateToken, error handling
- `src/api/auth/route.ts` - Integrated token validation

**New Files:**
- `src/types/auth.ts` - Auth type definitions

**Todos:** {X}/{Y} complete
```

**If `{auto_mode}` = true:**
→ Proceed to validation

**If `{auto_mode}` = false:**

Use `ask_followup_question`:
```
Implementation complete. Ready to validate?

Options:
- Proceed to validation (Recommended): Run typecheck, lint, and tests
- Review changes: I want to review what was changed
- Make adjustments: I want to modify something
```

### 7. Complete Save Output (if save_mode)

**If `{save_mode}` = true:**

Append to `{output_dir}/03-execute.md`:
```markdown
---
## Step Complete
**Status:** ✓ Complete
**Files modified:** {count}
**Todos completed:** {count}
**Next:** step-04-validate.md
**Timestamp:** {ISO timestamp}
```

---

## SUCCESS METRICS:

✅ All plan items implemented
✅ All todos marked complete
✅ No scope creep - only plan items
✅ Files read before modification
✅ Typecheck and lint pass
✅ Progress logged (if save_mode)

## FAILURE MODES:

❌ Adding features not in the plan
❌ Modifying files without reading first
❌ Not updating todos as you work
❌ Multiple todos in_progress simultaneously
❌ Ignoring type or lint errors
❌ **CRITICAL**: Not using `ask_followup_question` for blockers

## EXECUTION PROTOCOLS:

- Follow the plan EXACTLY
- Read before write
- One file at a time
- Update todos in real-time
- Fix errors immediately

---

## NEXT STEP:

After implementation complete, load `./step-04-validate.md`

<critical>
Remember: Execution is about following the plan - don't redesign or add features!
</critical>
