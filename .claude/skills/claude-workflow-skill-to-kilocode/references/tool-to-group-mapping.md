# Tool to Group Mapping

Complete mapping of Claude Code tools to Kilocode groups.

## Core Mapping Table

| Claude Code Tool | Kilocode Group | Notes |
|-----------------|----------------|-------|
| `Read` | `read` | File reading |
| `Glob` | `read` | File pattern matching |
| `Grep` | `read` | Content searching |
| `Edit` | `edit` | File modification |
| `Write` | `edit` | File creation |
| `MultiEdit` | `edit` | Multiple edits |
| `NotebookEdit` | `edit` | Jupyter notebooks |
| `Bash` | `command` | Shell commands |
| `WebFetch` | `browser` | URL fetching |
| `WebSearch` | `browser` | Web searching |
| `MCP` | `mcp` | MCP server tools |
| `Task` | `mcp` | Subagent delegation |

## Kilocode Groups Reference

| Group | Capabilities |
|-------|-------------|
| `read` | Read files, search content, find patterns |
| `edit` | Create, modify, delete files |
| `browser` | Web access, URL fetching, search |
| `command` | Execute shell commands |
| `mcp` | Access MCP servers and tools |

## Conversion Logic

```python
def map_tools_to_groups(tools: list[str]) -> list[str]:
    mapping = {
        'Read': 'read',
        'Glob': 'read',
        'Grep': 'read',
        'Edit': 'edit',
        'Write': 'edit',
        'MultiEdit': 'edit',
        'NotebookEdit': 'edit',
        'Bash': 'command',
        'WebFetch': 'browser',
        'WebSearch': 'browser',
        'MCP': 'mcp',
        'Task': 'mcp',
    }

    groups = set()
    for tool in tools:
        tool_clean = tool.strip()
        if tool_clean in mapping:
            groups.add(mapping[tool_clean])

    return sorted(list(groups))
```

## Special Cases

### No Tools Specified

If Claude Code agent has no `tools` field:
- Agent inherits all tools
- Convert to: all groups (`read`, `edit`, `command`, `browser`, `mcp`)

### Tool with Patterns

Claude Code allows tool patterns like `Bash(git *)`:
- Extract base tool name: `Bash`
- Map to group: `command`
- Note: Kilocode doesn't support tool-level restrictions

### Disallowed Tools

If `disallowedTools` is specified:
- Start with all groups
- Remove groups for disallowed tools
- Example: `disallowedTools: Write, Edit` → exclude `edit` group

## Built-in Agent Mappings

| Claude Code Agent | Default Groups |
|-------------------|----------------|
| `Explore` | `read` |
| `Plan` | `read` |
| `general-purpose` | `read`, `edit`, `command`, `browser`, `mcp` |
| `Bash` | `command` |
| `Snipper` | `read`, `edit` |

## Examples

### Example 1: Read-only Agent
```yaml
# Claude Code
tools: Read, Grep, Glob

# Kilocode
groups:
  - read
```

### Example 2: Code Modifier
```yaml
# Claude Code
tools: Read, Edit, Write, Bash

# Kilocode
groups:
  - read
  - edit
  - command
```

### Example 3: Full Access
```yaml
# Claude Code
tools: (not specified - inherits all)

# Kilocode
groups:
  - read
  - edit
  - browser
  - command
  - mcp
```

### Example 4: Browser Agent
```yaml
# Claude Code
tools: WebFetch, WebSearch, Read

# Kilocode
groups:
  - read
  - browser
```
