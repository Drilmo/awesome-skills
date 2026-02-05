# Kilocode Mode Schema Reference

## Complete Field Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `slug` | String | Yes | Unique identifier. Pattern: `/^[a-zA-Z0-9-]+$/` |
| `name` | String | Yes | Display name with optional emoji |
| `description` | String | Yes | Short summary for mode selector |
| `roleDefinition` | String | Yes | Detailed role, expertise, and personality |
| `groups` | Array | Yes | Tool access configuration |
| `whenToUse` | String | No | Guidance for automated mode selection |
| `customInstructions` | String | No | Additional behavioral guidelines |

## Groups (Tool Access)

Basic access:
```yaml
groups:
  - read
  - edit
  - browser
  - command
  - mcp
```

With file restrictions:
```yaml
groups:
  - read
  - [edit, {fileRegex: '\.md$', description: "Markdown files only"}]
```

## Storage Locations

**Project-level** (highest priority):
- File: `.kilocodemodes` in project root
- Format: YAML or JSON

**Global**:
- File: `custom_modes.yaml` (or `.json`)
- Location: Kilocode config directory

**Mode-specific rules**:
- Directory: `.kilo/rules-{slug}/`
- Fallback: `.kilorules-{slug}` file

## Examples

### Documentation Writer
```yaml
customModes:
  - slug: docs-writer
    name: "📝 Documentation Writer"
    description: Technical documentation specialist
    roleDefinition: |
      You are a technical writer specializing in clear, comprehensive documentation.
      Focus on clarity, examples, and proper structure.
    groups:
      - read
      - [edit, {fileRegex: '\.md$', description: "Markdown files only"}]
    customInstructions: |
      - Use clear headings and bullet points
      - Include code examples where relevant
      - Keep language accessible
```

### Code Reviewer
```yaml
customModes:
  - slug: code-reviewer
    name: "🔍 Code Reviewer"
    description: In-depth code review and analysis
    roleDefinition: |
      You are an experienced code reviewer focused on quality, security, and best practices.
      Identify bugs, security issues, and suggest improvements.
    groups:
      - read
    whenToUse: When reviewing pull requests or auditing code quality
    customInstructions: |
      - Check for security vulnerabilities
      - Verify error handling
      - Suggest performance improvements
```

### Full-Stack Developer
```yaml
customModes:
  - slug: fullstack-dev
    name: "🚀 Full-Stack Developer"
    description: Full-stack development with all tools
    roleDefinition: |
      You are a senior full-stack developer proficient in modern web technologies.
      Build complete features from database to UI.
    groups:
      - read
      - edit
      - browser
      - command
      - mcp
```

### Test Engineer
```yaml
customModes:
  - slug: test-engineer
    name: "🧪 Test Engineer"
    description: Testing and quality assurance specialist
    roleDefinition: |
      You are a QA engineer focused on comprehensive testing.
      Write unit tests, integration tests, and ensure code coverage.
    groups:
      - read
      - [edit, {fileRegex: '\.(test|spec)\.(ts|js|tsx|jsx)$', description: "Test files only"}]
      - command
    customInstructions: |
      - Aim for high code coverage
      - Test edge cases and error paths
      - Use descriptive test names
```
