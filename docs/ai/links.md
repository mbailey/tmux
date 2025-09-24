# tmux Official Documentation Links

## Primary Documentation

- **tmux Manual Page**: https://man7.org/linux/man-pages/man1/tmux.1.html
  - Comprehensive reference for all tmux commands and options
  - Includes pane numbering behavior documentation

- **tmux GitHub Wiki**: https://github.com/tmux/tmux/wiki
  - Getting Started guide
  - Advanced configuration examples
  - Community-contributed content

## Key Concepts

### Pane Numbering Behavior
From the official manual: "Pane numbers are not fixed, instead panes are numbered by their position in the window, so if the pane with number 0 is swapped with the pane with number 1, the numbers are swapped as well as the panes themselves."

The numbering follows a left-to-right, top-to-bottom priority system:
- Leftmost panes get lower numbers
- Among panes at the same horizontal position, topmost get lower numbers
- Numbers are reassigned after any split or rearrangement

## Additional Resources

- **tmux Cheat Sheet**: https://tmuxcheatsheet.com/
  - Quick reference for common commands
  - Visual layout examples

- **tmux Source Code**: https://github.com/tmux/tmux
  - Official repository
  - Change logs and release notes