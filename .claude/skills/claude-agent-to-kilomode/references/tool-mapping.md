# Tool Mapping Reference

Complete mapping from Claude Code agent tools to Kilocode mode groups.

## Tool to Group Mapping

### `read` Group
Grants file reading capabilities.

| Claude Code Tool | Notes |
|-----------------|-------|
| Read | Direct file reading |
| Glob | File pattern matching |
| Grep | Content search |
| LS (via Bash) | Directory listing |

### `edit` Group
Grants file modification capabilities.

| Claude Code Tool | Notes |
|-----------------|-------|
| Edit | String replacement edits |
| Write | Full file creation/overwrite |
| MultiEdit | Multiple edits in one call |
| NotebookEdit | Jupyter notebook editing |

**File restrictions**: Can limit with `fileRegex`:
```yaml
- [edit, {fileRegex: '\.md$', description: "Markdown only"}]
- [edit, {fileRegex: '\.(test|spec)\.(ts|js)$', description: "Test files only"}]
```

### `command` Group
Grants shell command execution.

| Claude Code Tool | Notes |
|-----------------|-------|
| Bash | Shell command execution |

### `browser` Group
Grants web browsing capabilities.

| Claude Code Tool | Notes |
|-----------------|-------|
| WebFetch | URL content fetching |
| WebSearch | Web search queries |

### `mcp` Group
Grants MCP (Model Context Protocol) tool access.

| Claude Code Tool | Notes |
|-----------------|-------|
| MCP tools | External tool integrations |
| Task | Subagent spawning |

## Emoji Inference Guide

| Agent Purpose | Suggested Emoji |
|--------------|-----------------|
| Code review | 🔍 |
| Fast/speed | ⚡ |
| Documentation | 📝 |
| Testing | 🧪 |
| Security | 🔒 |
| Search/explore | 🔎 |
| Build/create | 🏗️ |
| Fix/debug | 🔧 |
| Clean/refactor | ✨ |
| Web/browser | 🌐 |

## Edge Cases

### Agent with no tools specified
Default to `read` only for safety.

### Agent with "All tools" or "*"
Map to all groups: `read`, `edit`, `browser`, `command`, `mcp`.

### Agent with Skill tool
The Skill tool has no direct equivalent. Include skill behavior in `customInstructions`.

### Agent with model preference
Kilocode modes don't have model selection. Note in output for user awareness.
