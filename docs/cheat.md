# TMUX CHEAT SHEET - PREFIX: Ctrl+b

## SESSIONS

d    Detach
$    Rename
s    List all
L    Last session (toggle)
(    Previous session
)    Next session

## WINDOWS

c    Create
,    Rename
&    Kill
w    List all
f    Find
n    Next
p    Previous
l    Last window (toggle)
0-9  Go to number

## PANES

-      Split horizontal
|      Split vertical
←↑→↓   Switch pane
C-←↑→↓ Resize pane
z      Zoom toggle
x      Kill
{      Swap left
}      Swap right
q      Show numbers
;      Last pane (toggle)

## COPY MODE (PREFIX + [)

Space  Start selection
Enter  Copy selection
/      Search forward
?      Search backward
n      Next match
N      Previous match

## OTHER

r    Reload config
/    This cheat sheet  ← you are here
T    Toggle light/dark theme
?    List all keybindings
:    Command prompt

## REMOTE / NESTED TMUX

You're in TWO tmux layers when you mosh into a remote tmux:
  outer  = local tmux on this machine
  inner  = remote tmux (e.g. `mosh ms2 -- tmux attach -t work`)

Both layers eat Ctrl-b. To send a prefix to the INNER tmux:
  C-b C-b    Send prefix through to inner layer
             then release, then press the inner command key

  Example: detach from inner session
    C-b C-b d   →  detaches inner (you stay in outer)
    C-b   d     →  detaches outer (closes whole local session)

Mosh sandboxes $TMUX, so the "sessions should be nested with care"
warning does NOT fire on mosh-into-tmux. If you DO see that warning,
you've started a plain `tmux` inside an existing local pane — exit it
or `unset TMUX; tmux` to force.

Tip: to avoid the prefix collision entirely, set the REMOTE tmux
prefix to C-a (in ms2's tmux.conf) and leave local as C-b.
