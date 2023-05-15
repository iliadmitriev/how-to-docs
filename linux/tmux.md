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

