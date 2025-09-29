# Tmux Theme Switching

## Overview

This tmux configuration provides automatic theme switching that follows your operating system's appearance (light/dark mode) across macOS, WSL, and Linux distributions.

## How It Works

1. **Automatic Detection**: The `tmux-theme-detect` script detects your OS appearance:
   - **macOS**: Reads AppleInterfaceStyle from system defaults
   - **WSL**: Queries Windows registry via PowerShell for theme setting
   - **Linux**: Checks gsettings (GNOME/GTK), KDE, or XFCE settings

2. **Theme Application**: The `tmux-theme-apply` script:
   - Detects current OS theme (or reads manual override)
   - Applies appropriate tmux theme (light.conf or dark.conf)
   - Updates all running tmux sessions

3. **Auto-loading**: Your tmux.conf automatically:
   - Sources the saved theme on startup
   - Applies the correct theme based on OS detection

## Manual Control

### Toggle Theme
Use `prefix + T` (usually `Ctrl-b T`) to manually toggle between light and dark themes. This creates a manual override that persists until removed.

### Remove Manual Override
```bash
rm ~/.terminal-theme
tmux-theme-apply
```

### Force Specific Theme
```bash
echo "dark" > ~/.terminal-theme
tmux-theme-apply
# or
echo "light" > ~/.terminal-theme
tmux-theme-apply
```

## Platform-Specific Notes

### macOS
- Automatically detects system appearance changes
- Works with both Intel and Apple Silicon Macs
- Theme detection is instant

### WSL (Windows Subsystem for Linux)
- Detects Windows 10/11 app theme setting
- Requires PowerShell to be accessible from WSL
- Alternative: Set `WINDOWS_THEME` environment variable

### Linux (Ubuntu/Fedora)
- Supports GNOME/GTK (via gsettings)
- Supports KDE Plasma (via kreadconfig5)
- Supports XFCE (via xfconf-query)
- Falls back to dark theme if detection fails

## Customizing Themes

Edit the theme files to customize colors:
- Dark theme: `tmux/config/themes/dark.conf`
- Light theme: `tmux/config/themes/light.conf`

After editing, reload your tmux configuration:
```bash
tmux source-file ~/.tmux.conf
```

Or use the reload keybinding: `prefix + r`

## Environment Variables

You can override theme detection using environment variables:

```bash
# For WSL when PowerShell detection doesn't work
export WINDOWS_THEME="dark"  # or "light"

# Generic fallback for any system
export TERMINAL_THEME="dark"  # or "light"
```

## Troubleshooting

### Theme not applying automatically
1. Check that scripts are in PATH:
   ```bash
   which tmux-theme-detect
   which tmux-theme-apply
   ```

2. Test theme detection:
   ```bash
   tmux-theme-detect
   ```

3. Manually apply theme:
   ```bash
   tmux-theme-apply
   ```

### WSL theme detection not working
If PowerShell detection fails in WSL, you can:
1. Set the `WINDOWS_THEME` environment variable in your shell config
2. Use manual theme control with `prefix + T`
3. Create a scheduled task in Windows to update a file that WSL can read

### Theme changes not persisting
Make sure `~/.tmux.theme` is writable and that your tmux.conf includes:
```tmux
if-shell 'test -f ~/.tmux.theme' 'source-file ~/.tmux.theme'
```