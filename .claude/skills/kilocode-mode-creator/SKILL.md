---
name: kilocode-mode-creator
description: Create custom modes for Kilocode. Use when user asks to "create a mode", "add a kilocode mode", "new mode for kilocode", or wants to define AI behavior modes. Supports -a flag for automatic mode generation.
argument-hint: "[-a] <mode description>"
---

# Kilocode Mode Creator

Create custom modes for Kilocode with proper structure and validation.

## Argument Parsing

Parse arguments to detect:
- `-a` flag: Auto mode (no questions, generate based on description)
- Everything else: Mode description/requirements

## Workflow

### If `-a` flag is present (Auto Mode)

Generate the mode immediately based on the description provided. Infer all fields intelligently:

1. Parse the description to extract mode purpose
2. Generate all required fields:
   - `slug`: kebab-case identifier derived from name
   - `name`: Display name with appropriate emoji
   - `description`: Short summary
   - `roleDefinition`: Detailed role description
   - `groups`: Appropriate tool access based on mode purpose
3. Add optional fields if relevant:
   - `whenToUse`: If the mode has specific trigger conditions
   - `customInstructions`: If special behaviors are needed
4. Output the YAML configuration

### If `-a` flag is NOT present (Interactive Mode)

Ask clarifying questions for anything not specified. Questions to consider:

1. **Mode purpose**: What should this mode do? What's its specialty?
2. **Name and emoji**: What should it be called? Any preferred emoji?
3. **Tool access**: Which capabilities does it need?
   - `read` - Read files
   - `edit` - Edit files (can restrict with fileRegex)
   - `browser` - Web browsing
   - `command` - Run commands
   - `mcp` - MCP tools
4. **File restrictions**: Should editing be limited to specific file types?
5. **Behavior guidelines**: Any specific instructions or personality traits?

Ask only about what's missing or unclear. Don't ask about things already specified.

## Output Format

Generate YAML format for `.kilocodemodes` (project) or `custom_modes.yaml` (global):

```yaml
customModes:
  - slug: mode-slug
    name: "Emoji Mode Name"
    description: Short description for mode selector
    roleDefinition: |
      Detailed description of the mode's role, expertise, and personality.
      Can be multiple lines for comprehensive role definition.
    groups:
      - read
      - [edit, {fileRegex: '\\.ext$', description: "File type restriction"}]
      - browser
      - command
      - mcp
    whenToUse: Optional guidance for automated mode selection
    customInstructions: |
      Optional additional behavioral guidelines.
```

## Validation Rules

- **slug**: Only letters, numbers, hyphens. Must be unique.
- **fileRegex in YAML**: Single backslash (`\.md$`)
- **fileRegex in JSON**: Double backslash (`\\.md$`)

## Reference

See [references/mode-schema.md](references/mode-schema.md) for complete field documentation and examples.
