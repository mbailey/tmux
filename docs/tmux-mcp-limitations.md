# tmux MCP Limitations

## Issue: Cannot Execute tmux Control Commands

**Date**: 2025-01-21
**MCP Server**: tmux-mcp via npx

### Problem

The tmux MCP `execute-command` tool is designed to run commands *inside* tmux panes, not to execute tmux control commands themselves.

When attempting to create a split pane:
```javascript
mcp__tmux__execute-command({
  paneId: "%19",
  command: "tmux split-window -h"
})
```

This results in the error:
```
command new-session: unknown flag -h
```

The MCP server appears to be misinterpreting the command or running it in an unexpected context.

### Root Cause

The `execute-command` tool sends keystrokes to the specified pane as if you typed them. It's meant for running shell commands inside the pane, not for controlling tmux itself.

Commands like `split-window`, `new-window`, `kill-pane` etc. are tmux control commands that need to be executed by tmux directly, not typed into a shell.

### Workaround

For now, tmux control operations must be performed manually:
- Split horizontally: `Ctrl-b %`
- Split vertically: `Ctrl-b "`
- New window: `Ctrl-b c`
- Navigate panes: `Ctrl-b arrow`

### Potential Solutions

1. **Extend MCP**: Add new tools like `split-pane`, `create-window` that execute tmux commands directly
2. **Use Bash tool**: Run tmux commands via the Bash tool instead
3. **Different MCP server**: Look for a more full-featured tmux MCP implementation

### Related
- tmux MCP GitHub: https://github.com/zcaceres/tmux-mcp
- MCP specification: https://modelcontextprotocol.io