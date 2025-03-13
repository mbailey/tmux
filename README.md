# tmux

## Creating a named session
To create a new named tmux session:

```bash
tmux new -s session-name
```

You can also create a new session in detached state:

```bash
tmux new -s session-name -d
```

## Context-Aware Cheat Sheets

The `?` key binding will show a context-aware cheat sheet:
- When using tmux normally, it shows the tmux cheat sheet
- When in a program like vim, git, etc., it shows that program's cheat sheet if available

This works by detecting the current pane's command and looking for a corresponding cheat sheet at:
`~/public/PROGRAM/docs/cheat.md`

Supported programs:
- tmux (default)
- vim/nvim
- git
- (add more by updating the mapping in the tmux-cheat script)

## See also

- [clipboard](https://github.com/tmux/tmux/wiki/Clipboard#quick-summary)
