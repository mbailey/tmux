# tmux-dev-window Specification

## Overview

`tmux-dev-window` creates a new tmux window with a specialized layout for development work, combining an editor, AI assistant, and shell in a productive three-pane configuration.

## Purpose

Provide a single command to set up a development environment with:
- Editor for coding (left pane)
- AI assistant for help (right pane)  
- Shell for commands (bottom pane)

## Command Interface

### Usage

```bash
tmux-dev-window [OPTIONS] [PATH]
```

### Arguments

- `PATH` - File or directory to open in the editor (optional, defaults to current directory with preferred file selection)

### Options

- `-h, --help` - Show usage information
- `-w, --window-name NAME` - Custom name for the tmux window (overrides all naming strategies)
- `-s, --naming-strategy STRATEGY` - Naming strategy: directory|git-root|filename|smart (default: directory)
- `--git-root` - Use git repository name for window name (shorthand for --naming-strategy git-root)
- `-e, --editor COMMAND` - Editor command (default: $EDITOR or nvim)
- `-a, --ai-command COMMAND` - AI assistant command (default: $AI_COMMAND or claude)
- `-t, --target-session SESSION` - Target tmux session (default: current session)
- `-n, --no-switch` - Create window but don't switch to it
- `-d, --dry-run` - Show commands that would be executed without running them

### Environment Variables

- `TMUX_DEV_EDITOR` - Default editor command (overrides $EDITOR)
- `TMUX_DEV_AI_COMMAND` - Default AI assistant command  
- `TMUX_DEV_LAYOUT_PRESET` - Layout preset: "default", "wide-editor", "wide-ai" (default: "default")
- `TMUX_DEV_NAMING_STRATEGY` - Default naming strategy: directory|git-root|filename|smart (default: directory)
- `TMUX_DEV_WINDOW_NAME` - Default window name (overrides strategy if set)
- `NO_COLOR` - Disable colored output when present

## Layout Configuration

### Default Layout (50/50 top, 25% bottom)

```
+-------------------+-------------------+
|                   |                   |
|     Editor        |    AI Assistant   |
|   (50% width)     |   (50% width)     |
|                   |                   |
|                   |                   |
+-------------------+-------------------+
|               Shell                   |
|           (100% width)                |
+---------------------------------------+
```

### Wide Editor Layout (60/40 top, 25% bottom)

```
+-------------------------+-------------+
|                         |             |
|        Editor           | AI Assistant|
|     (60% width)         | (40% width) |
|                         |             |
|                         |             |
+-------------------------+-------------+
|               Shell                   |
|           (100% width)                |
+---------------------------------------+
```

### Wide AI Layout (40/60 top, 25% bottom)

```
+-------------+-------------------------+
|             |                         |
|   Editor    |    AI Assistant         |
| (40% width) |     (60% width)         |
|             |                         |
|             |                         |
+-------------+-------------------------+
|               Shell                   |
|           (100% width)                |
+---------------------------------------+
```

## Behavior

### Window Creation

1. If no PATH provided, uses current directory with preferred file selection
2. Validates PATH argument exists (if provided)
3. Resolves to absolute path
4. Determines target directory and editor target:
   - If PATH is a file: use parent directory, open the file
   - If PATH is a directory: use that directory, select preferred file
   - If no PATH: use current directory, select preferred file
5. Generates window name using specified strategy (see Window Naming section)
6. Creates new tmux window with the determined name
7. Sets up three-pane layout according to preset

### Preferred File Selection

When opening a directory (or no path provided), the editor will attempt to open files in this order of preference:

1. `./docs/tasks/README.md` (relative to target directory)
2. `./README.md` (relative to target directory)  
3. `REPO_ROOT/docs/tasks/README.md` (if in a git repository)
4. `REPO_ROOT/README.md` (if in a git repository)
5. The directory itself (if no preferred files found)

### Pane Setup

1. **Left Pane (Editor)**:
   - Changes to target directory
   - Launches editor with PATH argument

2. **Right Pane (AI Assistant)**:
   - Changes to target directory  
   - Launches AI command

3. **Bottom Pane (Shell)**:
   - Changes to target directory
   - Remains at shell prompt

### Error Handling

- Exit with error if not inside tmux session (unless -t specified)
- Validate PATH exists before creating window
- Check editor and AI commands exist
- Handle tmux command failures gracefully
- Provide clear error messages with suggested fixes

### Edge Cases

- Handle paths with spaces and special characters
- Work with both relative and absolute paths
- Support symlinks (resolve to actual path)
- Handle read-only directories appropriately
- Work when called from different working directories

## Implementation Details

### Script Structure

```bash
#!/usr/bin/env bash
set -o nounset -o pipefail -o errexit

# Constants and defaults
readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"
readonly DEFAULT_EDITOR="${TMUX_DEV_EDITOR:-${EDITOR:-nvim}}"
readonly DEFAULT_AI_COMMAND="${TMUX_DEV_AI_COMMAND:-${AI_COMMAND:-claude}}"
readonly DEFAULT_LAYOUT="${TMUX_DEV_LAYOUT_PRESET:-default}"

# Color support (respecting NO_COLOR)
if [[ -n "${NO_COLOR:-}" ]] || [[ ! -t 1 ]]; then
    readonly RED="" GREEN="" YELLOW="" BLUE="" RESET=""
else
    readonly RED=$'\033[0;31m' GREEN=$'\033[0;32m' 
    readonly YELLOW=$'\033[0;33m' BLUE=$'\033[0;34m' 
    readonly RESET=$'\033[0m'
fi
```

### Key Functions

1. **usage()** - Display help information
2. **error()** - Print error message and exit
3. **validate_path()** - Check if path exists and is accessible
4. **validate_command()** - Check if command exists in PATH
5. **get_window_name()** - Generate window name using specified strategy
6. **get_directory_name()** - Directory-based naming (default strategy)
7. **get_git_root_name()** - Git repository-based naming
8. **sanitize_window_name()** - Clean window name for tmux compatibility
9. **create_layout()** - Set up the three-pane layout
10. **setup_panes()** - Send commands to panes

### Tmux Commands Used

- `tmux new-window` - Create new window
- `tmux split-window` - Create pane splits
- `tmux select-layout` - Apply layout preset
- `tmux send-keys` - Send commands to panes
- `tmux select-pane` - Switch between panes
- `tmux rename-window` - Set window name

## Testing Considerations

### Unit Tests

- Path validation with various inputs
- Command existence checking
- Window name generation with all strategies
- Name sanitization edge cases
- Git root detection functionality
- README.md special handling
- Argument parsing

### Integration Tests

- Window creation in active session
- Layout dimensions verification
- Pane command execution
- Error handling with missing commands
- Different layout presets

### Manual Testing Checklist

- [ ] Creates window with file path
- [ ] Creates window with directory path
- [ ] Handles paths with spaces
- [ ] Respects custom editor command
- [ ] Respects custom AI command
- [ ] Layout presets work correctly
- [ ] Window naming strategies work correctly
- [ ] Never uses "README.md" as window name
- [ ] Git root naming works in repositories
- [ ] Custom window names override strategies
- [ ] Error messages are clear
- [ ] Help text is accurate
- [ ] Works from any directory
- [ ] Handles missing dependencies gracefully

## Window Naming

### Naming Strategies

#### Directory Strategy (Default)
- For files: uses parent directory name
- For directories: uses directory name
- Never uses "README.md" as window name (uses parent directory instead)
- Examples:
  - `/projects/myapp/src/main.py` → `myapp`
  - `/projects/myapp/README.md` → `myapp` 
  - `/projects/myapp/` → `myapp`

#### Git Root Strategy
- Uses git repository root name when available
- Falls back to directory strategy if not in git repository
- Examples:
  - `/projects/myapp/nested/deep/file.py` (in git repo "myapp") → `myapp`
  - `/tmp/script.py` (not in git) → `tmp`

#### Filename Strategy
- Uses filename for files (without extension), directory name for directories
- Exception: README.md always uses parent directory
- Examples:
  - `/projects/myapp/server.py` → `server`
  - `/projects/myapp/README.md` → `myapp`
  - `/projects/myapp/` → `myapp`

#### Smart Strategy
- Uses contextual logic based on path patterns
- Recognizes common project structures (packages/PROJECT, src/PROJECT)
- Falls back to git root or directory name
- Examples:
  - `/code/mt-public/packages/tmux/bin/script` → `tmux`
  - `/projects/myapp/src/components/` → `myapp`

### Name Sanitization
- Replaces non-alphanumeric characters with underscores
- Ensures tmux-compatible window names
- Preserves readability while maintaining functionality

### Override Options
```bash
# Custom window name (highest priority)
tmux-dev-window -w "custom-name" /path/to/file

# Different naming strategy
tmux-dev-window --naming-strategy git-root /path/to/file
tmux-dev-window --git-root /path/to/file  # shorthand

# Environment variable defaults
export TMUX_DEV_NAMING_STRATEGY=git-root
export TMUX_DEV_WINDOW_NAME=my-dev-window  # always use this name
```

## Security Considerations

- Sanitize PATH input to prevent command injection
- Don't execute arbitrary commands from environment variables
- Validate all user inputs before using in tmux commands
- Use proper quoting for all variables
- Sanitize window names to prevent tmux command injection

## Future Enhancements

1. **Layout Customization**:
   - Allow custom split percentages via options
   - Support horizontal vs vertical main split
   - Save/load layout configurations

2. **Session Management**:
   - Option to create in new session
   - Integration with tmux-resurrect for persistence
   - Project-specific configurations

3. **Tool Integration**:
   - Auto-detect project type and adjust tools
   - Support different AI assistants
   - Configure based on file type

4. **Productivity Features**:
   - Auto-focus editor pane on creation
   - Keyboard shortcuts for pane navigation
   - Status line showing current file/directory

## Dependencies

- tmux (>= 2.0)
- bash (>= 4.0)
- Editor command (nvim recommended)
- AI assistant command (claude or similar)
- Standard unix tools: basename, dirname, realpath

## Related Documentation

- [tmux package README](../../README.md)
- [Bash conventions](../../../../../conventions/languages/bash.md)
- [CLI design standards](../../../../../conventions/interfaces/cli.md)
- [tmux documentation](https://github.com/tmux/tmux/wiki)