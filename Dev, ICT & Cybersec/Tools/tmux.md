---
Description: tmux’s authors describe it as a terminal multiplexer. Behind this fancy term hides a simple concept, that within one terminal window you can open multiple windows and split-views (called “panes” in tmux lingo). Each pane will contain its own, independently running terminal instance. This allows you to have multiple terminal commands and applications running visually next to each other without the need to open multiple terminal emulator windows.
URL: https://github.com/tmux/tmux/wiki
---

### Tmux parameters

| Command | Description |
| --- | --- |
| tmux ls | List all running sessions |
| tmux attach -t <number> | Attach the <number> session |
| tmux a #<name> | Attach the <name> session |
| tmux kill-session #<name> | Kill the <name> session |
| tmux new -s <name> | Give a name to the new created session |
| tmux rename-session -t <number> <new_name> | Rename the session <number> to <new_name> |

### Traditional Mode:

**prefix key** + **command key**
By default, tmux uses `C-b` as prefix key. (`C-` means “press and hold the `Ctrl` key").

My PrefKey is `C-f`

### Sessions shortcuts

| Shortcut | Description |
| --- | --- |
| PrefKey + :new | New session |
| PrefKey + s | List all sessions |
| PrefKey + $ | Rename current session |
| PrefKey + d | Detach current session |

### Windows shortcuts

| Shortcut | Description |
| --- | --- |
| PrefKey + c | Create new window |
| PrefKey + w | List all windows |
| PrefKey + p (ALT+PagDown) | Switch to the previous window |
| PrefKey + n (ALT+PagUp) | Switch to the next window |
| PrefKey + <number> | Switch to <number> window |
| PrefKey + f | Find window |
| PrefKey + , | Rename the current windows |
| PrefKey + & | Kill the current windows |
| PrefKey + :swap-window -t -1 | Swap window with the previous one |
| PrefKey + :swap-window -s <X> -t <Y> | Swap window number <X> with the window number <Y> |

### Panes shortcuts

| Shortcut | Description |
| --- | --- |
| PrefKey + % ( | ) | Split vertically |
| PrefKey + " ( - ) | Split horizontally |
| PrefKey + <Arrow> (ALT+<arrow>) | Switch pane |
| PrefKey + Ctrl + o | Swap panes |
| PrefKey + q | Show pane number |
| PrefKey + x | Kill current pane |
| PrefKey + z | Make a pane go full-screen / shrink it back to its previous size |
| PrefKey + space | Swap window's panes layout |
| PrefKey + } | Move the current pane left |
| PrefKey + { | Move the current pane left |
| PrefKey + Ctrl + <arrow> | Resize pane in <arrow> direction |
| PrefKey + Alt + <arrow> | Resize pane in <arrow> direction (more space) |
| PrefKey + ! | Break current pane in a separate window |

### Navigation Mode shortcuts

| Shortcut | Description |
| --- | --- |
| PrefKey + [ | Enable navigation mode |
| ? | Search up |
| / | Search down |
| n | Search again |
| ^ | Back to indentation |
| Space | Start selection |
| Esc | Clear selection |
| Enter | Copy selection |
| p | Paste buffer |
| j (<arrow down>) | Cursor down |
| h (<arrow left>) | Cursor left |
| l (<arrow right>) | Cursor right |
| k (<arrow up>) | Cursor up |
| H | Cursor to top line |
| M | Cursor to middle line |
| L | Cursor to bottom line |
| $ | End of line |
| 0 | Start of line |
| : | Go to line |
| Ctrl + d | Half page down |
| Ctrl + u | Half page up |
| Ctrl + f | Next page |
| Ctrl + b | Previous page |
| w | Next word |
| b | Previous word |
| q | Quit mode |
| Ctrl + <arrow down> (J) | Scroll down |
| Ctrl + <arrow up> (K) | Scroll up |
| d | Delete entire line |
| D | Delete to end of line |

### Misc shortcuts

| Shortcut | Description |
| --- | --- |
| PrefKey + ? | List all commands |
| PrefKey + t | Big clock |
| PrefKey + : | Prompt for commands |

### Custom Configuration:

This is my current configuration. It was created to:

- declare more convenient and easily memorized shortcuts
- implement custom functionality to more conveniently handle clipboard and buffer saving
- show current IPs in the bottom bar so you don't have to search for them every time
- implement the use of the mouse

```bash
# === Unbind keys ===

#unbind system defined prefix
unbind C-b

# === Bind keys to prefix table ===

# vertical split
bind -N "Vertical split" | split-window -h -c '#{pane_current_path}'

# horizontal split
bind -N "Horizzontal split" - split-window -v -c '#{pane_current_path}'

# source .tmux.conf file
bind -N "Reload tmux.conf file" r source-file /etc/tmux.conf \; display "Configuration Reloaded!"

# Change prefix
bind C-f send-prefix

# Send tmux buffer to system clipboards
bind -N "Copy tmux clipboard to system primary" P run "tmux show-buffer | xclip -sel p -i >/dev/null"\; display "Tmux clipboard sent to system clipboard!"
bind -N "Copy tmux clipboard to system clipboard" C run "tmux show-buffer | xclip -sel c -i >/dev/null"\; display "Tmux clipboard sent to system primary!"
bind -N "Copy tmux clipboard to all clipboard" a run "tmux show-buffer | xclip -sel p -i -f | xclip -sel c -i >/dev/null"\; display "Tmux clipboard sent to all clipboards!"

# Paste tmux clipboard using Ctrl-F + p
bind -N "Paste the most recent tmux buffer" p paste-buffer

# === Bind keys to root table ===

# switch panes using Alt-arrow without prefix
bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

# switch windows using Alt+Pag without prefix
bind -n M-PPage previous-window
bind -n M-NPage next-window

# === Set options  ===

# Vi mode for Copy mode
setw -g mode-keys vi

# Send tmux clipboard to system clipboard when pressing y in copy-mode
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard'

# set prefix to control-f
set -g prefix C-f

# helps in faster key repetition
set -sg escape-time 0

# don't allow tmux to rename the window based on commands running
set-window-option -g allow-rename off

# Enable mouse and copy from mouse selection
set -g mouse on        # For tmux version 2.1 and up

# show host name and IP address on left side of status bar
set -g status-right-length 300
set -g status-right  "#[fg=#7400ff] #([ -d /proc/sys/net/ipv4/conf/eth0 ] && ip addr show dev eth0 | grep "inet[^6]" | awk '{print \"eth0: \" $2}') #[fg=#ff00ff] #([ -d /proc/sys/net/ipv4/conf/wlan0 ] && ip addr show dev wlan0 | grep "inet[^6]" | awk '{print \"wlan0: \" $2}') #[fg=#d70000] #([ -d /proc/sys/net/ipv4/conf/tun0 ] && ip addr show dev tun0 | grep "inet[^6]" | awk '{print \"tun0: \" $2}') #[fg=black] \"#{=22:pane_title}\" %H:%M %d-%b-%y"
```