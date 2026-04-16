# Sending Input to Claude Code via tmux

How to reliably send text to a running Claude Code TUI session from
another pane, with accurate notes on the known gotchas.

## The Basic Pattern

Text and Enter must be issued as **separate** `tmux send-keys` commands.
Chaining them via `;` or combining them into one invocation risks the
Enter being consumed by Claude Code's paste handler (see below).

```bash
# RIGHT -- two separate tmux invocations
tmux send-keys -t "$target" -l "your message here"
tmux send-keys -t "$target" Enter

# WRONG -- text and Enter in one call does not reliably submit
tmux send-keys -t "$target" "your message here" Enter
```

The `-l` (literal) flag is required to send the text verbatim without
keyword interpretation (e.g. so the word `Enter` in your message stays
as text rather than becoming a key press).

**Prefer `agents minion send`** when available -- it wraps this pattern
and handles verification.

## Finding the Target Pane

Inside tmux, `$TMUX_PANE` holds the pane ID of the shell that inherited
the environment -- **your own** pane. To target a sibling pane (e.g. the
pane below you running a Claude Code session):

```bash
sibling=$(tmux list-panes -F '#{pane_id}' | grep -v "^${TMUX_PANE}\$")
tmux send-keys -t "$sibling" -l "hello"
tmux send-keys -t "$sibling" Enter
```

`#P` and `tmux display-message -p '#{pane_id}'` without a target return
the **active** pane of the current window, which is not necessarily
your pane. Always use `$TMUX_PANE` when you mean "myself."

## Verifying the Message Landed

After sending, capture the target pane and check:

```bash
tmux capture-pane -p -t "$sibling" | tail -20
```

Look for your message in the prompt area. If you see
`[Pasted text #N +M lines]`, see the next section.

## The Bracketed-Paste Placeholder

Claude Code treats any single stdin chunk larger than **800 characters**
(the `PASTE_THRESHOLD` constant) as a bracketed paste. Rather than
rendering each character, it displays a compact placeholder in the
prompt:

```
❯ [Pasted text #1 +42 lines]
```

This is purely a display choice. The content is still in the prompt and
a subsequent Enter submits it normally. `[Pasted text]` does **not**
block submission on its own.

What triggers the 800-char threshold via tmux:

- `tmux paste-buffer` delivers the whole buffer in one write -- easily
  crosses the threshold for any non-trivial message.
- `tmux send-keys -l` streams characters through the pty, usually in
  small batches. Even multi-kilobyte strings rarely cross the threshold
  in a single input event, so they normally render inline without the
  placeholder.

## The Real Gotcha: Same-Chunk Race

The Claude Code paste handler (`src/hooks/usePasteHandler.ts`) has this
documented race, excerpted from the source comment:

> When paste + a keystroke arrive in the same stdin chunk, both
> `wrappedOnInput` calls run in the same `discreteUpdates` batch before
> React commits -- the second call reads stale `pasteState.timeoutId`
> (null) and takes the `onInput` path. If that key is Enter, it
> submits the old input and the paste is lost.

A `pastePendingRef` was added to mitigate it, but in practice the
observable failure mode is:

**The Enter is swallowed, the paste accumulates, the message stays in
the prompt as `[Pasted text #N +M lines]` and no submit occurs.**

### Reproducing It

Chain paste-buffer and Enter in a single tmux command so both arrive in
the same stdin chunk:

```bash
# FAILS -- Enter is swallowed, placeholder accumulates
tmux paste-buffer -b msg -p -t "$target" \; send-keys -t "$target" Enter
```

Observed behaviour after running the above five times in quick
succession:

```
❯ [Pasted text #6 +13 lines][Pasted text #7 +13 lines][Pasted text #8 +13 lines]...
```

None submitted.

### Avoiding It

Issue `paste-buffer` / `send-keys -l` and `send-keys Enter` as
**separate tmux commands**, not chained:

```bash
# WORKS -- paste delivered first, Enter arrives in a later chunk
tmux paste-buffer -b msg -p -t "$target"
tmux send-keys -t "$target" Enter

# Also works
tmux send-keys -t "$target" -l "$multi_line_msg"
tmux send-keys -t "$target" Enter
```

No sleep is required in practice -- the shell dispatch between
consecutive tmux invocations already separates the pty writes enough
that node sees them as distinct input events. A small `sleep 0.1`
between paste and Enter is a cheap insurance policy if you've seen
flakiness in your environment.

## Summary Cheat Sheet

| Want                               | Do                                      |
| ---------------------------------- | --------------------------------------- |
| Find your own pane                 | `$TMUX_PANE`                            |
| Find a sibling pane                | `tmux list-panes -F '#{pane_id}' \| grep -v "^$TMUX_PANE$"` |
| Send short text                    | `send-keys -l` then separate `Enter`    |
| Send multi-line / large text       | `load-buffer` + `paste-buffer` then separate `Enter` |
| Avoid the swallowed-Enter race     | Do not chain paste + Enter with `\;`    |
| Verify delivery                    | `tmux capture-pane -p -t "$target"`     |
