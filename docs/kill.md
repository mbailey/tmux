# kill tmux server

To check if a tmux server is still running:

```bash
ps aux | grep tmux
```

or more specifically:

```bash
pgrep -f tmux
```

## Killing the tmux Server

To completely stop the tmux server:

```bash
tmux kill-server
```
