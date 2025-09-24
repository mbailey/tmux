# tmux MCP Server Documentation

The tmux Model Context Protocol (MCP) server enables AI assistants to interact with tmux sessions programmatically. This document provides comprehensive information about setup, usage, and best practices.

## Overview

The tmux MCP server provides tools for:
- Managing tmux sessions, windows, and panes
- Executing commands in specific panes
- Capturing pane content
- Automating terminal workflows

## Available tmux MCP Servers

Here are the available tmux MCP server implementations (as of 2025-07-20):

### 1. **nickgnd/tmux-mcp** (Recommended)
- **GitHub**: https://github.com/nickgnd/tmux-mcp
- **Stars**: Check current count on GitHub
- **Package**: `tmux-mcp`
- **Description**: A mature MCP server for tmux with comprehensive features
- **Installation**: `npm install -g tmux-mcp`

### 2. **lox/tmux-mcp-server**
- **GitHub**: https://github.com/lox/tmux-mcp-server
- **Stars**: Check current count on GitHub
- **Description**: An MCP server that lets AI agents interact with terminal sessions through tmux
- **Installation**: See repository for details

### 3. **jonrad/tmux-mcp**
- **GitHub**: https://github.com/jonrad/tmux-mcp
- **Stars**: Check current count on GitHub
- **Description**: Control your tmux session via AI - POC implementation
- **Installation**: See repository for details

### 4. **kesor/mcp-server-tmux**
- **GitHub**: Repository may have moved or been renamed
- **Package**: `@kesor/mcp-server-tmux`
- **Description**: Alternative tmux MCP implementation
- **Note**: Verify current availability before use

## Installation

### Using Claude Code (Recommended)

```bash
# For the recommended server (nickgnd/tmux-mcp)
claude mcp add tmux --scope user -- npx -y tmux-mcp

# With shell type configuration (optional)
claude mcp add tmux --scope user -- npx -y tmux-mcp --shell-type=fish
```

### Manual Installation

1. Install the server:
```bash
# Recommended
npm install -g tmux-mcp

# Alternative
npm install -g @kesor/mcp-server-tmux
```

2. Add to your Claude configuration (`~/claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "tmux": {
      "command": "npx",
      "args": ["@kesor/mcp-server-tmux"]
    }
  }
}
```

## Available Tools

### Session Management
- `list-sessions` - List all tmux sessions
- `find-session` - Find session by name or ID
- `create-session` - Create a new tmux session

### Window Management
- `list-windows` - List windows in a session
- `create-window` - Create a new window

### Pane Management
- `list-panes` - List panes in a window
- `capture-pane` - Capture content from a pane
- `execute-command` - Execute a command in a specific pane
- `get-command-result` - Get the result of a command execution

## Best Practices for AI Agents

### 1. Always Read tmux Configuration First

Before using tmux commands, **always read ~/.tmux.conf** to understand:
- Window numbering (may start at 1 instead of 0)
- Custom key bindings
- Pane navigation settings
- User-specific configurations

### 2. Use Dedicated Command Panes

**Critical Rule**: Always execute commands in a dedicated command pane, never in panes running interactive programs.

#### Why This Matters
The tmux MCP server sends keystrokes to panes. If a pane is running:
- **Editors (vim, neovim)**: Commands become text input or trigger editor commands
- **Interactive programs**: Input gets consumed by the program
- **File browsers (NeoTree)**: May create files with command text as names

#### Standard Pane Layout
- **Command Bar** (bottom): Dedicated command execution pane (often %4)
- **Editor Left** (top-left): Primary editor/viewer pane (often %2)
- **Editor Right** (top-right): Secondary editor/viewer pane (often %3)

Note: Pane IDs change with each session, always verify current IDs.

### 3. File Editing Protocol

When making changes to files:
1. Edit the file using appropriate tools
2. Open the file in a top pane (left or right) after editing
3. Navigate to and highlight the changed section
4. Show the user what was modified before proceeding
5. Use visual mode or cursor positioning to indicate changes

### 4. Verification Protocol

After executing any tmux command:
1. Use `capture-pane` to check the result
2. Verify the command executed as expected
3. Look for error messages or unexpected behavior
4. If something went wrong, acknowledge and correct it

This is especially important for:
- File navigation commands
- Opening files in editors
- Any command that changes system state

## Usage Examples

### Basic Command Execution

```javascript
// Bad: Executing in active editor pane
await tmux.execute_command({ pane: "%3", command: "ls -la" });  // May insert text into editor!

// Good: Execute in dedicated command pane
await tmux.execute_command({ pane: "%4", command: "ls -la" });  // Clean command execution
```

### Workflow Example

1. List sessions and windows:
```javascript
const sessions = await tmux.list_sessions();
const windows = await tmux.list_windows({ session: "main" });
```

2. Find the command pane:
```javascript
const panes = await tmux.list_panes({ session: "main", window: "1" });
const commandPane = panes.find(p => p.current_command === "bash" || p.current_command === "zsh");
```

3. Execute commands:
```javascript
await tmux.execute_command({ 
  pane: commandPane.id, 
  command: "cd ~/projects && git status" 
});
```

4. Verify execution:
```javascript
const output = await tmux.capture_pane({ pane: commandPane.id });
console.log(output);
```

## Integration with Neovim

When using tmux with Neovim MCP:
- The `NVIM_SOCKET_PATH` environment variable is set per window
- This allows the Neovim MCP server to target the correct editor instance
- Always check for this variable when working with editor panes

## Common Pitfalls to Avoid

1. **Don't execute in wrong panes**: Always verify pane type before execution
2. **Don't assume pane IDs**: They change between sessions
3. **Don't skip verification**: Always check command results
4. **Don't forget context**: Read tmux.conf and understand user's setup

## Configuration Options

### Shell Type
The tmux MCP server can be configured with different shell types:
```bash
# Default (bash)
npx -y tmux-mcp

# Fish shell
npx -y tmux-mcp --shell-type=fish

# Zsh
npx -y tmux-mcp --shell-type=zsh
```

The shell type is used to properly read command exit status.

### Environment Variables
No specific environment variables are required for tmux MCP servers. The server uses standard tmux commands and the tmux socket for communication.

## See Also

- [AI Best Practices](../AI.md) - General tmux best practices for AI agents
- [tmux Clipboard Integration](ai/tmux-clipboard.md) - Working with tmux clipboard
- [Development Window Setup](tmux-dev-window.md) - Using tmux-dev-window script

## Additional Resources

- Official tmux documentation: https://github.com/tmux/tmux/wiki
- MCP Protocol specification: https://modelcontextprotocol.org/