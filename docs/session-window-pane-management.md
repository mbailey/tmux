# tmux Session, Window, and Pane Management

## Sessions

### List Sessions
```bash
tmux ls                              # List all sessions
tmux list-sessions                   # Same as above
tmux ls -F '#{session_name}'         # List session names only
```

### Attach to Sessions
```bash
tmux attach                          # Attach to most recent session
tmux attach -t session-name          # Attach to specific session
tmux a -t 5                          # Attach to session 5
```

### Create Sessions
```bash
tmux new -s session-name             # Create and attach to new session
tmux new -s session-name -d          # Create session in detached state
tmux new -s dev -c ~/projects        # Create session in specific directory
```

### Kill Sessions
```bash
tmux kill-session -t session-name    # Kill specific session
tmux kill-session -a                 # Kill all sessions except current
tmux kill-session -a -t keep-this    # Kill all except specified session
```

### Rename Sessions
```bash
tmux rename-session -t old new       # Rename session
# Inside tmux: Ctrl+b $              # Rename current session
```

## Windows

### List Windows
```bash
tmux list-windows                    # List windows in current session
tmux list-windows -t session-name    # List windows in specific session
tmux lsw -F '#{window_name}'         # List window names only
```

### Create Windows
```bash
tmux new-window                      # Create window in current session
tmux new-window -n window-name       # Create named window
tmux new-window -t session:          # Create window in specific session
```

### Switch Windows
```bash
# Inside tmux:
# Ctrl+b n                           # Next window
# Ctrl+b p                           # Previous window
# Ctrl+b 0-9                         # Switch to window by number
# Ctrl+b w                           # Choose window from list
```

### Kill Windows
```bash
tmux kill-window -t window-number    # Kill specific window
tmux killw -t session:window         # Kill window in specific session
# Inside tmux: Ctrl+b &              # Kill current window
```

### Rename Windows
```bash
tmux rename-window new-name          # Rename current window
tmux renamew -t 3 new-name           # Rename specific window
# Inside tmux: Ctrl+b ,              # Rename current window
```

## Panes

### List Panes
```bash
tmux list-panes                      # List panes in current window
tmux list-panes -t session:window    # List panes in specific window
tmux lsp -F '#{pane_id}'             # List pane IDs
```

### Identifying Your Own Pane

When running inside tmux, `$TMUX_PANE` is set to the pane ID of the shell
(or program) that inherited the environment. This is the pane you are in
-- not necessarily the active pane of the window.

```bash
echo "$TMUX_PANE"                    # e.g. %42 -- the pane YOU are in
tmux display-message -p '#{pane_id}' # Active pane's ID (may differ)
tmux display-message -p '#P'         # Active pane's index (may differ)
```

**Why this matters for AI agents and scripts:** `display-message` and `#P`
report the active pane of the current window, which may not be the pane
your script is running in. To reliably reference "self," use `$TMUX_PANE`.

```bash
# Self-aware capture (works even if another pane is active)
tmux capture-pane -p -t "$TMUX_PANE"

# Target the other pane in a two-pane window (assumes self + one sibling)
sibling=$(tmux list-panes -F '#{pane_id}' | grep -v "^${TMUX_PANE}\$")
tmux send-keys -t "$sibling" -l "message"
tmux send-keys -t "$sibling" Enter
```


### Create Panes
```bash
tmux split-window -h                 # Split horizontally (side by side)
tmux split-window -v                 # Split vertically (top/bottom)
tmux splitw -h -p 30                 # Split with 30% width
tmux splitw -v -c ~/projects         # Split in specific directory
# Inside tmux:
# Ctrl+b %                           # Split horizontally
# Ctrl+b "                           # Split vertically
```

### Navigate Panes
```bash
# Inside tmux:
# Ctrl+b arrow-keys                  # Move between panes
# Ctrl+b q                           # Show pane numbers, press number to switch
# Ctrl+b o                           # Cycle through panes
```

### Resize Panes
```bash
# Inside tmux:
# Ctrl+b Ctrl+arrow                  # Resize pane
# Ctrl+b Alt+arrow                   # Resize by 5 cells
tmux resize-pane -L 10               # Resize left by 10 cells
tmux resize-pane -R 10               # Resize right by 10 cells
tmux resize-pane -U 5                # Resize up by 5 cells
tmux resize-pane -D 5                # Resize down by 5 cells
```

### Kill Panes
```bash
tmux kill-pane -t pane-id            # Kill specific pane
# Inside tmux: Ctrl+b x              # Kill current pane
```

### Send Commands to Panes
```bash
tmux send-keys -t session:window.pane 'command' Enter
tmux send-keys -t 0:1.0 'ls -la' Enter      # Send to session 0, window 1, pane 0
tmux send-keys -t %5 'clear' Enter          # Send to pane ID %5
```

## Environment Variables

### Show Environment
```bash
tmux show-environment                # Show all environment variables
tmux show-environment VAR_NAME       # Show specific variable
tmux show-environment -t window      # Show for specific window
```

### Set Environment
```bash
tmux set-environment VAR_NAME value  # Set variable
tmux setenv -t window VAR value      # Set for specific window
tmux set-environment -u VAR_NAME     # Unset variable
```

## Advanced Commands

### Get Information
```bash
tmux display-message -p '#S'         # Show current session name
tmux display -p '#{window_name}'     # Show current window name
tmux display -p '#{pane_current_command}'  # Show command in current pane
tmux info                            # Show all key bindings and commands
```

### Pipe Pane Output
```bash
tmux pipe-pane -t 1 'cat > output.log'     # Capture pane output to file
tmux pipe-pane -t 1                        # Stop capturing
```

### Move Windows/Panes
```bash
tmux move-window -t target-session         # Move window to another session
tmux join-pane -t target-window            # Move pane to another window
tmux break-pane                            # Convert pane to window
```

## Useful Format Strings

When using `-F` flag, you can use these format strings:

- `#{session_name}` - Session name
- `#{window_index}` - Window index
- `#{window_name}` - Window name
- `#{pane_index}` - Pane index within window
- `#{pane_id}` - Unique pane identifier (e.g., %0, %1)
- `#{pane_current_command}` - Command running in pane
- `#{pane_current_path}` - Current directory of pane

Example:
```bash
tmux list-panes -a -F '#{session_name}:#{window_index}.#{pane_index} #{pane_current_command}'
```