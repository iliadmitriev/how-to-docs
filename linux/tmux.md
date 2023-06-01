### Configure

options for `~/.tmux.conf`
```bash
# enable mouse
set-option -g mouse on

# lines of history to preserve
set-option -g history-limit 20000

# copy buffer on selection to system clipboard
bind-key -T copy-mode MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "pbcopy"
```

### Keys


* `Ctrl + b`  base key
* `Ctrl + b :` start typing command
* `Ctrl + b %` split window on 2 panes horizontally (left and right)
* `Ctrl + b "` split window on 2 panes vertically (top and bottom)
* `Ctrl + b c` create a new window
* `Ctrl + c d` detach from tmux


### Commands

List active sessions
```bash
tmux list-sessions
tmux ls
```
Attach to a session with name `0`
```bash
tmux attach-session -t 0
tmux attach -t 0
```
Try to attach to a session named `main` or create it if it doesn't exist
```bash
tmux new-session -A -s main
tmux new -As main
```