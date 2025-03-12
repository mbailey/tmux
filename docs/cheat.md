# TMUX CHEAT SHEET
PREFIX KEY: Ctrl+b

WINDOWS:
  c     Create window    n/p   Next/prev window
  ,     Rename window    w     List windows
  [0-9] Go to window     f     Find window

PANES:
  "     Split horiz      %     Split vertical  
  ←↑→↓  Switch pane      C-←↑→↓ Resize pane
  z     Toggle zoom      x     Kill pane
  {/}   Swap pane        q     Show pane numbers

SESSIONS:
  d     Detach           s     List sessions
  $     Rename           (/)   Prev/next session

COPY MODE (PREFIX + [):
  Space Start select     Enter Copy selection
  /     Search forward   ?     Search backward
  n     Next match       N     Previous match
  j/k   Down/up          h/l   Left/right

OTHER:
  ?     Show bindings    :     Command prompt
  t     Show time        ~     Show messages
