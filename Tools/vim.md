# Vim commands

`vimtutor`

## Vim Modes

Three main modes: Normal/Command, Insert, Visual.
Bottom line of terminal displays the current mode: -- <MODE> --

- **Normal mode**: is used for navigation, searching, and executing commands
  - `esc` hops from anymode back to normal
- Insert mode allows for text input and editing. To switch to insert mode
  - `i` swaps to insert mode
  - `esc` key exists the mode
- Visual mode allows you to select portions of text that you perform editing tasks on. You can use visual mode specific commands to select text in various ways or you may use your mouse to select the desired text. Once selected, there are visual mode specific editing commands you may execute. Once executed, the mode is automatically switched back to the normal/command mode.
  - `v` key swaps to visual mode
  - `esc` exists

#### Navigation

```shell
             ^
             k            
       < h       l >               
             j                     
             v
```

- `h`: Move Left
- `j`: Move Down
- `k`: Move Up
- `l`: Move Right
- `ctrl + d`: Scroll half way down a page
- `ctrl + u`: Scroll half way up a page

#### Editing
- `x`: Delete a character 
- `i`: Insert text. 
  - Move cursor to character before the text to be inserted. 
  - hit `esc` to return to normal mode.
- `a`: Append
- `dd`: Delete current line
- `yy`: Yank (copy) current line
- `p`: Paste after the cursor
- `u`: Undo
- `Ctrl + r`: Redo


#### Editing file: Saving and Quitting

- `:wq` or `:x:` Save and quit
- `:w` Save changes
- `:q` Quit
- `:q!` Trash all changes





#### Searching

- `/<PATTERN_STRING>`: Search forward for a pattern. Note that this is case sensitive
- `?<PATTERN_STRING>`: Search backward for a pattern. Note that this is case sensitive
- `n`: Move to the next occurrence

