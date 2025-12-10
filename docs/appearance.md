# tmux Appearance

tmux theme configuration for automatic light/dark mode switching.

> Detailed theme switching documentation: [theme-switching.md](theme-switching.md)

## Quick Reference

### Check Current Theme
```bash
tmux-theme-detect  # Returns "light" or "dark"
```

### Apply Theme
```bash
tmux-theme-apply   # Applies theme based on OS detection
```

### Manual Toggle
Inside tmux: `prefix + T` (Ctrl-b T)

### Force Specific Theme
```bash
echo "dark" > ~/.terminal-theme && tmux-theme-apply
echo "light" > ~/.terminal-theme && tmux-theme-apply
```

### Return to Auto-Detection
```bash
rm ~/.terminal-theme && tmux-theme-apply
```

## Theme Files

Located in `config/themes/`:
- `dark.conf` - Catppuccin Mocha colors
- `light.conf` - Catppuccin Latte colors

## How It Works

1. `tmux-theme-detect` checks OS appearance (macOS defaults, gsettings, or Windows registry)
2. `tmux-theme-apply` sources the appropriate theme file
3. Theme is saved to `~/.tmux.theme` for persistence
4. Manual override via `~/.terminal-theme` takes precedence

## Reload Configuration
```bash
tmux source-file ~/.tmux.conf
# Or inside tmux: prefix + r
```

## See Also

- [appearance package](../../appearance/README.md) - Central appearance management
- [appearance skill](../../appearance/SKILL.md) - AI-assisted appearance configuration
- [theme-switching.md](theme-switching.md) - Detailed theme switching documentation
