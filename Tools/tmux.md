

## Tmux

- start session
  - `tmux`
  - `tmux new -s Session1`

- detatch from session via
  - `ctrl+b`(`ctrl-q` in my config file) then `D`

- kill sessions
  - `tmux kill-server`

- Split Window into 2 horizontal panes
  - `ctrl+q` -> `%`
- Split Window into 2 Vertical panes
  - `ctrl+q` -> `"`
- Move between panes
  - `ctrl+q` -> `Arrow key >/<`
- Close pane
  - `ctrl+q` -> `x`
- new window
  - `ctrl+q` -> `c`
- Move to next or previous pane
  - `ctrl+q` -> `n` | `p`
- Specify a window by number
  - `ctrl+q` -> `1,2,3..`
- Enter command line to type commands. Tab complete available
  - `ctrl+q` -> `:`
- View all key bindings. Press "Q" to exit
  - `ctrl+q` -> `?`
- Open a panel to navigate across windows in multiple sessions
  - `Ctrl+Q` -> `W` 