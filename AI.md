# TMUX Best Practices for AI Agents

## Important: Read Configuration First

**ALWAYS read ~/.tmux.conf before using tmux commands** to understand:
- Window numbering (may start at 1 instead of 0)
- Custom key bindings
- Pane navigation settings
- Any user-specific configurations

## Standard Pane Layout

When working with tmux-dev-window, use these standard pane positions:
- **Editor** (top-left): Primary editor pane (named "editor")
- **AI** (top-right): AI assistant pane (named "ai")
- **Shell** (bottom): Command execution pane (named "shell")

Note: Pane IDs change with each session, always verify current IDs.

## General tmux Guidelines

- Respect user's tmux configuration (base-index, pane-base-index)
- Use pane titles when available to identify pane purposes
- Be aware that users may have custom key bindings
- Consider window and pane numbering may start from 1, not 0

## tmux MCP Specific Guidelines

For detailed guidelines on using the tmux MCP server, see:
`packages/tmux/docs/tmux-mcp-ai.md`

## Benefits of Following These Practices

- Clear visibility of AI actions
- No interference with user's active work
- Clean command history
- Prevents accidental file creation or editor corruption

## Implementation Note

Consider maintaining awareness of the current tmux session context throughout your work session.