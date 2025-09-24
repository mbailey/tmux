# tmux MCP AI Agent Guidelines

Guidelines for AI agents using the tmux MCP server.

## Key Principle: Check Before Executing

**ALWAYS use `capture-pane` first** to check what's running in a pane before sending commands. Then execute commands only in panes with a shell prompt ready, not in panes running interactive programs.

## When to Use Which Tool

**Use the Bash tool for tmux control commands:**
- `tmux resize-pane`, `tmux select-pane`, `tmux split-window`
- `tmux list-windows`, `tmux new-window`, etc.
- Any command that controls tmux itself

**Use tmux MCP execute-command only for:**
- Sending text/commands to programs running inside panes
- Typing into editors, shells, or interactive programs
- When you need the command to appear as if typed by the user

## Known Limitations

**tmux MCP Cannot Execute Control Commands:**
- The `execute-command` tool sends keystrokes to panes, not tmux commands
- Attempting commands like `tmux split-window -h` will fail with errors
- This is a fundamental limitation of the current tmux MCP implementation
- For splits, new windows, etc., use manual keyboard shortcuts or the Bash tool

## Why This Matters

The tmux MCP execute-command sends keystrokes to the specified pane. If that pane is running:
- **Editors (vim, neovim)**: Commands become text input or trigger editor commands
- **Interactive programs**: Input gets consumed by the program
- **File browsers (NeoTree)**: May create files with command text as names

## Best Practice: Pane Safety Check

1. **Always check first**: Use `capture-pane` to see what's in a pane
2. **Look for shell prompts**: Only send commands to panes showing a shell prompt (e.g., `$` or `#`)
3. **Check pane titles**: If using tmux-dev-window, panes are named "editor", "ai", "shell"
4. **Prefer the shell pane**: The bottom "shell" pane is usually safest for commands
5. **Verify after execution**: Use `capture-pane` again to confirm the command ran

## Example Workflow

```bash
# WRONG: Using execute-command for tmux control
tmux execute-command -t %2 "tmux resize-pane -Z"  # NO! This types the command as text

# RIGHT: Using Bash tool for tmux control
bash: tmux resize-pane -Z -t %1  # Correct - controls tmux directly

# WRONG: Executing shell command in editor pane
tmux execute-command -t %1 "ls -la"  # NO! May insert text into editor

# RIGHT: Check pane first, then execute in shell pane
tmux capture-pane -t %3  # Check what's in the pane first
tmux execute-command -t %3 "ls -la"  # Execute only if it shows a shell prompt
```

## AI Agent File Editing Protocol

When making changes to files:
1. Always open the file in an editor pane after editing
2. Navigate to and highlight the changed section
3. Show the user what was modified before proceeding
4. Use visual mode or cursor positioning to indicate changes

## Verification Protocol

After executing any tmux command:
1. Use `capture-pane` to check the result
2. Verify the command executed as expected
3. Look for error messages or unexpected behavior
4. If something went wrong, acknowledge and correct it

This is especially important for:
- File navigation commands
- Opening files in editors
- Any command that changes system state

## Future Enhancements

Note: These guidelines will be integrated into our forked tmux MCP server to provide better safety and guidance. Future targets for similar patterns:
- VS Code integration (when needed)
- Other editor integrations as required