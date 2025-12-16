# tmux-dev-window

A tmux utility for creating productive development environments with a single command.

## Overview

`tmux-dev-window` creates a new tmux window with a specialized three-pane layout designed for development work:

- **Editor** (top-left): For writing code
- **AI Assistant** (top-right): For getting help and guidance
- **Shell** (bottom): For running commands

## Quick Start

```bash
# Create dev window in current directory
tmux-dev-window

# Create dev window for specific project
tmux-dev-window ~/projects/myapp

# Use custom AI command
tmux-dev-window -a "agents summon cora" .

# Custom window name
tmux-dev-window -w backend ~/backend/

# Get help
tmux-dev-window -h
```

## Installation

The script is located at `bin/tmux-dev-window` and should be in your PATH or called directly.

## Usage

```bash
tmux-dev-window [OPTIONS] [PATH]
```

### Arguments

- `PATH` - Target directory or file (optional, defaults to current directory)
  - If a file is provided, the window opens in that file's directory
  - Symlinks are automatically resolved

### Options

- `-w, --window NAME` - Window name (default: directory name)
- `-a, --ai CMD` - AI command to run (see Default AI Command below)
- `-t, --taskmaster` - Enable Taskmaster Console pane
- `-T, --no-taskmaster` - Disable Taskmaster Console pane
- `-h, --help` - Show help message

### Default AI Command

The AI command is determined in this order:
1. If `-a` flag is provided, use that
2. Else if `$TMUX_DEV_AI_CMD` is set, use that
3. Else if `agents` command exists, use `agents summon`
4. Else if `$AI_COMMAND` is set, use that (legacy)
5. Else use `claude`

### Taskmaster Console

The TMC pane is enabled when:
1. If `-t` flag is provided, enable it
2. If `-T` flag is provided, disable it
3. If `$TMUX_DEV_TMC` is set to `1` or `true`, enable it
4. If `tmc` command is available, auto-enable it

## Layout

The script creates a development layout:

### Standard (3-pane)
- **Editor** (top-left): 50% of top area
- **AI Assistant** (top-right): 50% of top area
- **Shell** (bottom): Full width, 25% of window height

### With Taskmaster (4-pane)
- **Editor** (top-left): 50% of top area (60% height)
- **AI Assistant** (top-right): 50% of top area (60% height)
- **Shell** (middle): Full width, 20% of window height
- **TMC** (bottom): Full width, 20% of window height

## Window Naming

By default, the window is named after the target directory. You can override this with the `-w` flag:

```bash
# Default: uses directory name
cd ~/projects/myapp
tmux-dev-window  # Window named "myapp"

# Custom name
tmux-dev-window -w backend  # Window named "backend"
```

If a window with the same name already exists, the script will switch to it instead of creating a new one.

## Environment Variables

Customize default behavior:

- `TMUX_DEV_AI_CMD` - Default AI command when not specified via -a flag
- `TMUX_DEV_TMC` - Enable TMC pane (set to `1` or `true` to enable)
- `EDITOR` - Editor command (defaults to 'nvim --listen /tmp/nvim-cora')
- `AI_COMMAND` - Legacy fallback for AI command (use TMUX_DEV_AI_CMD instead)

## Examples

```bash
# Basic usage - current directory
tmux-dev-window

# Specific directory
tmux-dev-window ~/projects/backend

# Open a specific file (uses file's directory)
tmux-dev-window ~/projects/backend/server.py

# Custom AI command
tmux-dev-window -a "agents summon cora" .

# Custom window name
tmux-dev-window -w api ~/projects/backend

# Set default AI command via environment
export TMUX_DEV_AI_CMD="agents summon vera"
tmux-dev-window

# Combine options
tmux-dev-window -w backend -a "agents summon cora" ~/projects

# Enable Taskmaster Console
tmux-dev-window -t .

# Disable Taskmaster Console (even if tmc is available)
tmux-dev-window -T .

# Enable TMC via environment
export TMUX_DEV_TMC=1
tmux-dev-window
```

## Compatibility

The script automatically adapts to your tmux configuration:

- Works with both 0-based and 1-based pane indexing
- Respects your default shell and editor settings
- Handles paths with spaces and special characters
- Works from any directory (uses absolute paths)
- Resolves symlinks automatically

## Requirements

- tmux (>= 2.0)
- bash (>= 4.0)
- Editor command (nvim recommended)
- AI assistant command (claude or similar)
- Standard unix tools: realpath

## Troubleshooting

### AI Command Not Found
If your AI command isn't available, the script will create the layout but show a warning. The AI pane will be created but the command may fail - you can manually start your AI tool.

### Wrong Layout
If panes appear in unexpected positions, check your tmux `pane-base-index` setting. The script automatically detects and adapts to both 0-based and 1-based indexing.

### Window Not Created
Ensure you're inside a tmux session or use the `-t` option to specify a target session.

## Related

- [tmux package README](../README.md) - Overview of all tmux utilities
- [tmux-dev-window spec](specs/tmux-dev-window.md) - Detailed implementation specification
- [tmux configuration](../config/dot-tmux.conf) - Recommended tmux settings