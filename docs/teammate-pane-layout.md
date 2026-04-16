# Teammate Pane Layout

A pattern for spawning multiple worker panes alongside a "leader" pane in the
same tmux window -- useful when you want to watch several agents (or
long-running jobs) work in parallel without losing sight of your primary
shell.

## The shape

```
+-------+------------+
|       |  worker-1  |
|       +------------+
|  LEA  |  worker-2  |
|  DER  +------------+
|       |  worker-3  |
|       +------------+
|       |  worker-4  |
+-------+------------+
  ~30%       ~70%
```

- Leader is pinned to the **left**, at a fixed percentage width (30% is a
  good default).
- Each new teammate is spawned on the **right**, then the whole window
  rebalances into `main-vertical` layout so teammates stack evenly.
- Leader keeps its width across rebalances; teammates grow thinner (in
  height) as more are added.

## Algorithm

### First teammate (window has only the leader)

```bash
tmux split-window -t "$LEADER_PANE" -h -l 70% -P -F '#{pane_id}'
```

- `-h` splits horizontally (new pane on the right)
- `-l 70%` makes the new pane 70% of the window width -- leader ends up at 30%
- `-P -F '#{pane_id}'` prints the new pane's id to stdout so you can target
  it for decoration (border colour, title, etc.)

No rebalance needed at this point -- the `-l 70%` already gives the right
split.

### Subsequent teammates (window has leader + N teammates)

```bash
# List all panes, drop the leader (first row), leaving just teammates
teammates="$(tmux list-panes -t "$WINDOW" -F '#{pane_id}' | tail -n +2)"
j=$(echo "$teammates" | grep -c .)       # teammate count
mid=$(( (j - 1) / 2 ))                   # middle teammate index
target="$(echo "$teammates" | sed -n "$((mid + 1))p")"

# Alternate vertical/horizontal splits based on teammate-count parity
if (( j % 2 == 1 )); then
  split_dir="-v"   # vertical split: new pane goes above/below target
else
  split_dir="-h"   # horizontal split: new pane goes left/right of target
fi

new_pane="$(tmux split-window -t "$target" "$split_dir" -P -F '#{pane_id}')"
```

This "middle-pane splitting with v/h alternation" produces a cascade -- new
teammates consistently split the middle of the existing teammate stack, so
the tree balances naturally as you add more.

### Rebalance after every spawn (when teammate count >= 2)

```bash
# Skip if there's only one teammate -- the initial 70% split is already right
pane_count="$(tmux list-panes -t "$WINDOW" -F '#{pane_id}' | wc -l)"
if [ "$pane_count" -gt 2 ]; then
  tmux select-layout -t "$WINDOW" main-vertical
  tmux resize-pane -t "$LEADER_PANE" -x 30%
fi
```

`main-vertical` puts the main pane (pane index 0 -- your leader) on the
left at `main-pane-width` columns, with all other panes stacked vertically
on the right. The follow-up `resize-pane -x 30%` overrides
`main-pane-width` with a percentage, which is usually what you want on a
variable-size window.

## Gotchas

### 1. `$TMUX_PANE` vs `display-message`

When your layout script spawns a new pane, tmux automatically focuses the
newly-created pane. On the **next** spawn, if you use
`tmux display-message -p '#{pane_id}'` to find your "leader," you'll get
the wrong pane -- you'll get the most recently spawned teammate.

**Use `$TMUX_PANE` instead.** It's the pane id of the shell (or script)
that inherited the environment -- stable for the lifetime of that process,
regardless of which pane is currently focused.

```bash
# WRONG -- returns focused pane, which shifts after each split-window
LEADER="$(tmux display-message -p '#{pane_id}')"

# RIGHT -- stable reference to the pane this script is running in
LEADER="${TMUX_PANE:?not running inside tmux}"
```

See `session-window-pane-management.md` for the broader self vs active pane
distinction.

### 2. `resize-pane -x <percent>` must come AFTER `select-layout`

Applying a layout (`main-vertical`, `tiled`, etc.) resets pane sizes. If
you run `resize-pane -x 30%` *before* `select-layout`, the layout will
overwrite your sizing. Always:

```bash
tmux select-layout -t "$WINDOW" main-vertical   # first
tmux resize-pane -t "$LEADER" -x 30%            # then
```

This is also why **manual user resizes are wiped every time a new teammate
is added** -- the layout call resets everything and the script reapplies
the leader width. If you need to preserve user resizes, skip the
rebalance conditionally (e.g. only rebalance on the first few spawns, or
when pane counts cross a threshold).

### 3. macOS BSD `sleep` doesn't accept `infinity`

A common trick for stub/daemon panes is `exec sleep infinity`. This works
on GNU coreutils but fails on macOS:

```
$ sleep infinity
usage: sleep number[unit] [...]
```

Portable alternatives:

```bash
exec sleep 86400            # one day -- fine for interactive sessions
exec tail -f /dev/null      # blocks until process is killed
```

### 4. `split-window` with a command argument

The command passed to `split-window` is run directly -- shell metacharacters
(`;`, `&&`, `|`) aren't interpreted unless you wrap in a shell:

```bash
# WRONG -- the ; is passed as a literal argument to printf
tmux split-window -h "printf 'hi\n'; exec sleep 86400"

# RIGHT -- wrap in sh -c
tmux split-window -h "sh -c \"printf 'hi\n'; exec sleep 86400\""
```

## Example: minion-pane

A working reference implementation lives at
[`ai-cora/minions/bin/minion-pane`](https://github.com/ai-cora/minions) (a
small bash script exposing `create`, `rebalance`, and `leader`
subcommands). It's wired into the project's `minion spawn --here` flag so
you can drop a worker pane into the current window with:

```bash
minion spawn my-worker --here --color colour2
```

## Credits

The cascading teammate pane layout documented here was observed in
**Claude Code** (Anthropic's official CLI for Claude) and codified as a
general-purpose pattern.

Credit to the Claude Code team for the design taste -- leader pinned on
the left, teammates cascading on the right, automatic rebalance after each
spawn is a lovely shape for multi-agent workflows. If you're using this
pattern with your own agents, you're walking a path Claude Code already
walked.
