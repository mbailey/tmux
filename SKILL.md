---
name: tmux
description: Terminal multiplexer management and workflow automation. Use this skill when working with tmux sessions, windows, and panes, managing terminal layouts, renaming windows intelligently based on project context, or automating terminal workflows with tmux commands.
---

# tmux Skill

Terminal multiplexer management with intelligent window naming and workflow automation.

## Purpose

This skill provides comprehensive tmux management capabilities including:
- Session, window, and pane management
- Intelligent window renaming based on project context
- Terminal workflow automation
- MCP server integration for AI-driven tmux control

## When to Use This Skill

Use this skill when:
- Managing tmux sessions, windows, or panes
- Renaming windows based on project or directory context
- Setting up development layouts with multiple panes
- Automating terminal workflows
- Integrating AI control with tmux via MCP
- Troubleshooting tmux configuration or layout issues

## Core Concepts

### Window Naming Strategy

Windows are automatically named based on these heuristics (in order of priority):

1. **Git Repository**: If all panes in a window are in the same git repository, use the repository name
2. **Common Path**: Use the common parent directory name shared by all panes
3. **Truncation**: Names are truncated to 20 characters maximum for readability
4. **Duplicate Handling**: Duplicate names get a suffix (-2, -3, etc.)
5. **Default Names Only**: Only rename windows with default command-based names (bash, node, python, etc.)
6. **Respect Custom Names**: Skip windows with user-assigned custom names
7. **Respect automatic-rename**: Skip windows where automatic-rename is enabled

### Standard Development Layout

When using `tmux-dev-window`, the standard pane layout is:
- **Editor** (top-left): Primary editor pane (named "editor")
- **AI** (top-right): AI assistant pane (named "ai")
- **Shell** (bottom): Command execution pane (named "shell")

## Tools and Scripts

### Window Management

#### `tmux-rename`

Intelligently rename tmux windows based on project context. (Previously called `tmux-rename-windows`.)

```bash
# Rename windows in current session (dry-run)
tmux-rename -n

# Actually rename windows
tmux-rename

# Target specific session
tmux-rename -s session-name

# Force rename even custom names
tmux-rename -f

# Verbose output
tmux-rename -v
```

**Behavior:**
- Analyzes all panes in each window to determine context
- Checks for git repositories and uses repo name when appropriate
- Falls back to common directory path if panes span multiple repos
- Skips windows with custom names (unless -f flag used)
- Skips windows with automatic-rename enabled
- Handles duplicate names by adding numeric suffixes

### Session and Layout Management

#### `tmux-dev-window`

Create a development window with standard three-pane layout.

```bash
# Create dev window in current session
tmux-dev-window

# Create in specific session
tmux-dev-window -s session-name
```

See `docs/tmux-dev-window.md` for detailed documentation.

#### `tmux-auto-connect`

Automatically connect to or create a tmux session.

```bash
# Auto-connect to default session
tmux-auto-connect

# Connect to specific session
tmux-auto-connect session-name
```

### Theme Management

#### `tmux-theme-detect` and `tmux-theme-apply`

Detect system appearance and apply matching tmux theme.

```bash
# Detect and apply theme automatically
tmux-theme-detect

# Apply specific theme
tmux-theme-apply light  # or dark
```

See `docs/theme-switching.md` for details.

### Interactive Tools

#### `tmux-cheat`

Display context-aware cheat sheets based on current pane's program.

```bash
# Show cheat sheet for current context
tmux-cheat

# In tmux, press ? to trigger this command
```

## Working with tmux Commands

### Essential Commands Reference

**Sessions:**
```bash
tmux ls                           # List sessions
tmux new -s name                  # Create named session
tmux attach -t name               # Attach to session
tmux rename-session -t old new    # Rename session
```

**Windows:**
```bash
tmux list-windows                 # List windows
tmux new-window -n name           # Create named window
tmux rename-window name           # Rename current window
tmux select-window -t :0          # Switch to window 0
```

**Panes:**
```bash
tmux list-panes                   # List panes
tmux split-window -h              # Split horizontally
tmux split-window -v              # Split vertically
tmux select-pane -t :.0           # Select pane 0
```

### Format Strings

When using tmux commands with `-F` flag:

- `#{session_name}` - Session name
- `#{window_index}` - Window index
- `#{window_name}` - Window name
- `#{pane_id}` - Unique pane identifier
- `#{pane_current_path}` - Current directory
- `#{pane_current_command}` - Running command

Example:
```bash
tmux list-panes -a -F '#{session_name}:#{window_index} #{window_name} #{pane_current_path}'
```

## AI Integration via MCP

### Reading Configuration First

**ALWAYS read `~/.tmux.conf` before using tmux commands** to understand:
- Window numbering (may start at 1 instead of 0)
- Custom key bindings
- Pane navigation settings
- User-specific configurations

### Using Dedicated Command Panes

**Critical Rule:** Always execute commands in dedicated command panes, never in panes running interactive programs.

**Why:** The tmux MCP server sends keystrokes. Commands sent to:
- Editors (vim, neovim): Become text input or trigger editor commands
- Interactive programs: Get consumed by the program
- File browsers: May create files with command text as names

**Standard Pane Identification:**
1. List panes with current command: `tmux list-panes -F '#{pane_id} #{pane_current_command}'`
2. Find panes running shells (bash, zsh, fish)
3. Use those panes for command execution

### Verification Protocol

After executing any tmux command via MCP:
1. Capture pane content to verify result
2. Check for errors or unexpected behavior
3. Confirm command executed as intended
4. Acknowledge and correct if something went wrong

See `docs/tmux-mcp.md` for comprehensive MCP integration documentation.

## Documentation Resources

Progressive disclosure documentation in `docs/`:

- `session-window-pane-management.md` - Comprehensive tmux command reference
- `tmux-mcp.md` - MCP server setup and AI integration guide
- `tmux-mcp-ai.md` - Best practices for AI agents using tmux MCP
- `tmux-dev-window.md` - Development window layout documentation
- `theme-switching.md` - Appearance-based theme switching
- `cheat.md` - Quick reference for tmux key bindings
- `clipboard.md` - Clipboard integration

## Configuration

### Metool Package Structure

This tmux package follows metool conventions:

- `bin/` - Executable scripts (symlinked to `~/.metool/bin/`)
- `shell/` - Shell functions, aliases, completions (sourced by metool)
- `config/` - Configuration files (symlinked to home directory)
- `docs/` - Documentation for progressive disclosure

See `~/Code/github.com/mbailey/metool/docs/conventions/package-structure.md` for details.

### Installation

```bash
# Install tmux package
mt install tmux

# This will:
# - Symlink bin/ scripts to ~/.metool/bin/
# - Source shell/ functions and aliases
# - Link config/ files to home directory
```

## Common Workflows

### Automatically Rename Windows

```bash
# Check what would be renamed (dry-run)
tmux-rename -n

# Apply the renames
tmux-rename

# Rename specific session
tmux-rename -s work
```

### Set Up Development Session

```bash
# Create new session with dev window
tmux new -s project
tmux-dev-window

# Or attach to existing session
tmux attach -t project
tmux-dev-window
```

### Quick Session Switch

```bash
# Auto-connect to session (creates if needed)
tmux-auto-connect work

# List all sessions
tmux ls

# Switch to specific session
tmux switch -t project
```

## Best Practices

### For AI Agents

1. **Always read `~/.tmux.conf` first** - Understand user configuration
2. **Verify pane types before execution** - Check what program is running
3. **Use dedicated shell panes** - Never execute in editor/program panes
4. **Capture and verify output** - Check command results
5. **Respect window names** - Don't rename custom-named windows without permission

### Sending Input to Claude Code Instances

When sending input to another Claude Code session via tmux, text and Enter must be sent as separate commands:

```bash
# WRONG - text and Enter in same call doesn't reliably submit
tmux send-keys -t pane "message" Enter

# RIGHT - send text with -l (literal) flag, then Enter separately
tmux send-keys -t pane -l "message"
tmux send-keys -t pane Enter
```

This is how `agents minion send` works internally. The `-l` flag ensures the message text is sent literally without interpretation. Always verify the message was received by checking the pane content after sending.

**Prefer using `agents minion send`** when available, as it handles this correctly:
```bash
agents minion send m-worker "your message here"
```

### For Users

1. **Name windows intentionally** - Custom names won't be auto-renamed
2. **Use standard layouts** - Consistent pane positions improve workflow
3. **Enable automatic-rename selectively** - For windows that should update automatically
4. **Run `tmux-rename` periodically** - Keep window names synchronized with projects

## Troubleshooting

### Windows Not Being Renamed

**Check:**
1. Does window have automatic-rename enabled? (`tmux show-window-options automatic-rename`)
2. Is window name custom (not a default command name)?
3. Run with verbose flag to see reasoning: `tmux-rename -n -v`

### Pane Commands Going to Wrong Location

**Solutions:**
1. List panes and their commands: `tmux list-panes -F '#{pane_id} #{pane_current_command}'`
2. Identify shell panes (bash, zsh, fish)
3. Target correct pane ID when sending commands
4. Read AI.md for best practices

### Session Layout Issues

**Check:**
1. Base-index setting: `tmux show-options -g base-index`
2. Pane-base-index: `tmux show-options -g pane-base-index`
3. Read user's ~/.tmux.conf for custom configurations

## See Also

- Official tmux wiki: https://github.com/tmux/tmux/wiki
- Metool documentation: `~/Code/github.com/mbailey/metool/`
- Model Context Protocol: https://modelcontextprotocol.org/
