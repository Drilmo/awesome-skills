# Ask Patterns

Patterns for using ask_followup_question tool effectively in workflow steps.

---

## Critical Rule

**NEVER use plain text prompts like:**
```
Choose an option:
1. Option A
2. Option B

Enter 1 or 2: _
```

**ALWAYS use ask_followup_question tool.**

---

## Basic Pattern

```
Question: "Full question ending with question mark?"
Options:
1. Option A (Recommended) - What this option does
2. Option B - What this option does
3. Option C - What this option does
```

For single selection: user picks one option
For multiple selection: user can pick multiple (specify in question)

---

## Common Patterns

### 1. Proceed/Cancel Pattern

```
Question: "Ready to continue with the next step?"
Options:
1. Continue (Recommended) - Proceed to the next phase
2. Review first - I want to review before continuing
3. Cancel - Stop the workflow
```

### 2. Approach Selection Pattern

```
Question: "Which approach should we use?"
Options:
1. Approach A (Recommended) - Description of approach A and tradeoffs
2. Approach B - Description of approach B and tradeoffs
3. Approach C - Description of approach C and tradeoffs
```

### 3. Scope Selection Pattern

```
Question: "How comprehensive should this be?"
Options:
1. Minimal - Only essential changes, fast
2. Balanced (Recommended) - Standard approach, good coverage
3. Comprehensive - Thorough coverage, takes longer
```

### 4. Yes/No Confirmation Pattern

```
Question: "Are you sure you want to proceed?"
Options:
1. Yes, proceed - Continue with the action
2. No, go back - Return to previous step
```

### 5. Multi-Select Features Pattern

```
Question: "Which features do you want to include? (Select all that apply)"
Options:
1. Feature A - Description of feature A
2. Feature B - Description of feature B
3. Feature C - Description of feature C
4. Feature D - Description of feature D
```

### 6. Error Recovery Pattern

```
Question: "An error occurred. How should we handle it?"
Options:
1. Retry - Try the operation again
2. Skip this step - Continue without completing this step
3. Debug manually - I'll investigate the issue
4. Abort workflow - Stop the entire workflow
```

### 7. Resolution Options Pattern

```
Question: "How would you like to handle these findings?"
Options:
1. Auto-fix all (Recommended) - Automatically fix all identified issues
2. Walk through each - Review and decide on each finding individually
3. Fix critical only - Only fix high-severity issues
4. Skip all - Acknowledge but don't make changes
```

### 8. Next Step Pattern

```
Question: "What would you like to do next?"
Options:
1. Continue to step X - Proceed with the next phase
2. Run optional step Y - Include optional step before continuing
3. Complete workflow - Finish here, skip remaining steps
```

---

## Auto Mode Handling

Always check auto_mode before asking:

```markdown
**If `{auto_mode}` = true:**
→ Use first/recommended option automatically

**If `{auto_mode}` = false:**
Use ask_followup_question:

Question: "..."
Options:
1. Option A (Recommended) - ...  # This is used in auto mode
2. Option B - ...
```

---

## Response Handling Pattern

After receiving user response:

```markdown
**Handle responses:**

**If "Option A":**
1. Do action A
2. Proceed to step X

**If "Option B":**
1. Do action B
2. Return to previous state

**If user provided custom input:**
1. Parse user input
2. Handle accordingly
```

---

## Guidelines

1. **Question**: Clear, specific, ends with "?"
2. **Options**: 2-4 options maximum
3. **Recommended**: Mark preferred option with "(Recommended)"
4. **Descriptions**: Brief but informative after the dash
5. **Multiple selection**: Specify in question when options aren't mutually exclusive
6. **Custom input**: Users can always provide custom input
